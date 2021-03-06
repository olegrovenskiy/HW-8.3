# HW-8.3

Финальный плейбук получился следующим:

    ---
    - name: Install Elasticsearch
      hosts: elasticsearch
      handlers:
        - name: restart Elasticsearch
          become: true
          service:
            name: elasticsearch
            state: restarted
      tasks:
        - name: "Download Elasticsearch's rpm"
          get_url:
            url: "https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-{{ elk_stack_version }}-x86_64.rpm"
            dest: "/tmp/elasticsearch-{{ elk_stack_version }}-x86_64.rpm"
          register: download_elastic
          until: download_elastic is succeeded
        - name: Install Elasticsearch
          become: true
          yum:
            name: "/tmp/elasticsearch-{{ elk_stack_version }}-x86_64.rpm"
            state: present
        - name: Configure Elasticsearch
          become: true
          template:
            src: elasticsearch.yml.j2
            dest: /etc/elasticsearch/elasticsearch.yml
          notify: restart Elasticsearch
    - name: Install Kibana
      hosts: kibana
      handlers:
        - name: restart Kibana
          become: true
          service:
            name: kibana
            state: restarted
      tasks:
        - name: "Download Kibana's rpm"
          get_url:
            url: "https://artifacts.elastic.co/downloads/kibana/kibana-{{ elk_stack_version }}-x86_64.rpm"
            dest: "/tmp/kibana-{{ elk_stack_version }}-x86_64.rpm"
          register: download_kibana
          until: download_kibana is succeeded
        - name: Install Kibana
          become: true
          yum:
            name: "/tmp/kibana-{{ elk_stack_version }}-x86_64.rpm"
            state: present
        - name: Configure Kibana
          become: true
          template:
            src: kibana.yml.j2
            dest: /etc/kibana/kibana.yml
          notify: restart Kibana
    - name: Install Filebeat
      hosts: app
      handlers:
        - name: restart Filebeat
          become: true
          service:
            name: filebeat
            state: restarted
      tasks:
        - name: "Download Filebeat's rpm"
          get_url:
            url: "https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-{{ elk_stack_version }}-x86_64.rpm"
            dest: "/tmp/filebeat-{{ elk_stack_version }}-x86_64.rpm"
          register: download_filebeat
          until: download_filebeat is succeeded
        - name: Install Filebeat
          become: true
          yum:
            name: "/tmp/filebeat-{{ elk_stack_version }}-x86_64.rpm"
            state: present
          notify: restart Filebeat   
        - name: Configure Filebeat
          become: true
          template:
            src: filebeat.yml.j2
            dest: /etc/filebeat/filebeat.yml
          notify: restart Filebeat
        - name: Set filebeat system work
          become: true
          command:
            cmd: filebeat modules enable system
            chdir: /usr/share/filebeat/bin
          register: filebeat_modules
          changed_when: filebeat_modules.stdout != 'Module system is already enabled'
        - name: Load Kibana dashboard
          become: true   
          command:
            cmd: filebeat setup
            chdir: /usr/share/filebeat/bin
          register: filebeat_setup
          changed_when: false
          until: filebeat_setup is succeeded
          
          
Плейбук содержит три однотипных плея для ElastickSearch? Filebeat and Kibana

1. Скачивание инсталяционного файла rpm, Elastic, Kibana, Filebeat https://www.elastic.co/
2. ELK версия 7.15.02 полследняя, прописана как переменная в ./inventory/prod/grpop_vars
3. Загрузка инсталяционного файла в /tmp/, загрузка проводится с проверкой усрешного выполнения
4. Установак пакетов и рестарт сервисов
5. теги не используются
6. Далее производится конфигурация сервисов в соответствии с файлами ./templater/<Service_Name>.yml.j2 
7. Виртуальные машины развёрнцты в yandex cloud
9. В плеях используется become для поднятия прав


