* Grafana erabiliz Datu bistaratzea
  Ondorengo jardueraren helburua, jadanik instalatuta egongo den grafana batetik abiatuta, Prometheus zerbitzari batek erakutsiko dituen datu espezifikoak bistaratzea izango da.
  
* Jardueraren berrerabilpena
  Jarduera hau IsardVDI bezalako ingurune birtualetan instalatzeko prestatu nahi da. Gainera ebaluaketa batetik bestera eta urte batetik bestera, hasierako egoera berrerabiltzeko asmoa dago.
  Gauzak horrela, **Ansible** erabiliko da eta konfigurazio bidezko instalazio eta martxan jartzea lortuko da.
  
* Prometheus eta Node Exporter
  Prometheus eta NOde Exporter zerbitzari batean egongo dira. Sare bidez atzigarri egongo dira gainontzeko makinentzako. Makina hauek izango lirateke jarduera aurrera eramateko tresnak eta berauetan egongo litzateke Grafana instalatuta.
  
  Zerbitzari honen instalazio eta konfigurazioa burutzeko, ondorengo Ansible fitxategiak erabili dira:
  
** inventory.ini
{% highlight  conf%}
[myhosts]
ansible_prometheus_node_exporter
{% endhighlight %}

Hemen gakoa host horretara pasahitz gabeko ssh gako bidezko sarrera konfiguratuta izatea da. Gainera ssh configurazio fitxategian **ansible_prometheus_node_exporter** hosta existitu beharko litzateke.

** node_exporter.yml
{% highlight  yaml%}
- hosts: myhosts
  become: True
  become_user: root
  vars:
    node_exporter_version: 1.1.2
  tasks:
    - name: download node exporter
      get_url:
        url: https://github.com/prometheus/node_exporter/releases/download/v{{ node_exporter_version }}/node_exporter-{{ node_exporter_version }}.linux-amd64.tar.gz
        dest: /tmp
    - name: unarchive node exporter
      unarchive:
        remote_src: yes
        src: /tmp/node_exporter-{{ node_exporter_version }}.linux-amd64.tar.gz
        dest: /tmp
    - name: move node exporter to /usr/local/bin
      copy:
        src: /tmp/node_exporter-{{ node_exporter_version }}.linux-amd64/node_exporter
        dest: /usr/local/bin/node_exporter
        remote_src: yes
        owner: root
        group: root
        mode: 0755
    - name: install unit file to systemd
      template:
        src: templates/node_exporter_service.j2
        dest: /etc/systemd/system/node_exporter.service
        owner: root
        group: root
        mode: 0600
    - name: configure systemd to use service
      systemd:
        daemon_reload: yes
        enabled: yes
        state: started
        name: node_exporter.service
{% endhighlight %}
Node Exporterren instalazio eta konfigurazioa.
** dhcp.yml
{% highlight  yaml%}
---
- name: Install and Configure ISC DHCP Server
  hosts: myhosts
  become: yes
  vars:
    # Network and IP range configuration
    dhcp_network: "192.168.85.0/24"
    dhcp_range_start: "192.168.85.20"
    dhcp_range_end: "192.168.85.40"
    dhcp_subnet_mask: "255.255.255.0"
    dhcp_routers: "192.168.85.6"  # Change this as necessary for your network
    dhcp_domain_name_servers: "8.8.8.8, 8.8.4.4"  # Use your DNS servers here

  tasks:
    - name: Install Kea
      ansible.builtin.package:
        name: 'kea-dhcp4-server'
        state: 'latest'
      register: _install_packages
      until: _install_packages is succeeded
      retries: 5
      delay: 2
      
    - name: Create Kea DHCP configuration file
      copy:
        dest: /etc/kea/kea-dhcp4.conf
        content: |
          {
            "Dhcp4":{
              "interfaces-config": {
                "interfaces": [
                "enp3s0"
                ]
                },
                "valid-lifetime": 28800,
                "subnet4": [
                {
                "subnet": "192.168.85.0/24",
                "pools": [
                {
                "pool": "192.168.85.20 - 192.168.85.40"
                }
                ],
                }
                ],
                "loggers": [
                {
                "name": "kea-dhcp4",
                "output_options": [
                {
                "output": "/tmp/kea-dhcp4.log",
                "maxver": 10
                }
                ],
                "severity": "INFO"
                }
                ]
                }
          }

        owner: root
        group: root
        mode: '0644'

    - name: Ensure Kea DHCP service is started and enabled
      systemd:
        name: kea-dhcp4-server
        state: started
        enabled: yes
{% endhighlight %}
Sare lokalean DHCP zerbitzari bat izatea beharrezkoa izango zaigu makina desberdin asko baditugu. Promeheus instalatuta dagoen makina berean instalatuko dugu DHCP4 zerbitzaria.
** netplan.yml
{% highlight  yaml%}
 - hosts: myhosts
   become: True
   become_user: root
   vars:
     netplan_remove_existing_configs: false
     netplan_configuration_dir: '/etc/netplan/'
     netplan_configuration_file: '3IA3.yaml'
     netplan_ethernets:
       - interface_name: 'enp3s0'
         dhcp4: 'no'
         addresses:
            - '192.168.85.6/24'
   roles:
         - role: hifis.toolkit.netplan
{% endhighlight %}
Erabiltzen ditugun IsardVDI irudiek Ubuntu daukate instalatuta eta netplan erabiltzen dute sarea konfiguratzeko. Prometheus zerbitzariari IP finkoa ezartzen zaio.

* Grafana
Grafana instalatzeko ere ansible erabiliko dugu. Horrela jardueran egin beharrekoa Grafanaren konfiguraziora mugatuko da.
** inventory.ini
{% highlight  conf%}
[myhosts]
ansible_grafana
{% endhighlight %}
inventoryari dakogionez aurreko ideia berdina. Makinak ssh gako bidezko eta pasahitz gabea aurre-ezarrita izan beharko du
{% highlight  yaml%}
- hosts: myhosts
  become: True
  become_user: root
  vars:
    grafana_admin_password: "abc1234"
  tasks:
    - name: update system
      ansible.builtin.apt:
        name: "*"
        state: latest
    - name: Import Grafana GPG signing key [Debian/Ubuntu]
      apt_key:
        url: "https://packages.grafana.com/gpg.key"
        state: present
        validate_certs: false
      register: _add_apt_key
      until: _add_apt_key is succeeded
      retries: 5
      delay: 2

    - name: Add Grafana repository [Debian/Ubuntu]
      apt_repository:
        repo: deb https://packages.grafana.com/oss/deb stable main
        state: present
        update_cache: true
      register: _update_apt_cache
      until: _update_apt_cache is succeeded
      retries: 5
      delay: 2

    - name: Install Grafana
      package:
        name: "grafana"
        state: "latest"
      register: _install_packages
      until: _install_packages is succeeded
      retries: 5
      delay: 2
    - name: start service grafana-server
      systemd:
        name: grafana-server
        state: started
        enabled: yes
    - name: wait for service up
      uri:
        url: "http://127.0.0.1:3000"
        status_code: 200
      register: __result
      until: __result.status == 200
      retries: 120
      delay: 1
    - name: change admin password for grafana gui
      shell : "grafana-cli admin reset-admin-password {{ grafana_admin_password }}"
      register: __command_admin
      changed_when: __command_admin.rc !=0
{% endhighlight %}
Aipatzekoa grafana admin erabiltzaileari jarri zaion pasahitza.
** netplan.yml
{% highlight  yaml%}
 - hosts: myhosts
   become: True
   become_user: root

   tasks:
    - name: Create Netplan configuration for enp3s0 interface with DHCP
      copy:
        dest: "/etc/netplan/00-dhcp-config.yaml"
        content: |
          network:
            version: 2
            renderer: networkd
            ethernets:
              enp3s0:
                dhcp4: true
                gateway4: 192.168.85.6
        owner: root
        group: root
        mode: '0600'

    - name: Apply Netplan configuration
      command: netplan apply
      become: true
{% endhighlight %}
Azkenik netplan konfiguratuko dugu kasu honetan DHCP bidez lortu dezan IP-a.
