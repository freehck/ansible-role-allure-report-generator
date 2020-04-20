freehck.allure_report_generator
=========

This role deploys a container with **allure_report_generator**

Description
-----------

**allure_report_generator** is a daemon that is monitoring `tests` directory, and run allure to generate reports over every new test directory it can find there. Generated reports are stored into `reports` directory, and can be used by Nginx or some other web server to provide graphical reports about build and integration tests.

Role Variables
--------------

Container variables:

- `allure_report_generator_ct_name`: container name, default is `"allure-report-generator"z
- `allure_report_generator_ct_image`: container image, default is `"freehck/allure-report-generator:2.13.2"`
- `allure_report_generator_ct_restart_policy`: default restart policy is `"always"`
- `allure_report_generator_ct_state`: default container state is `"started"`
- `allure_report_generator_ct_restart`: if we shall restart container, default is `false`
- `allure_report_generator_ct_pull`: if we shall try to pull the image every time we run deploy task, default is `true`
- `allure_report_generator_ct_recreate`: if we want to recreate container when we run deploy task, default is `false`

Data directories variables:

- `allure_report_generator_srv_dir`: default service dir, default is `"/srv/{{ allure_report_generator_ct_name }}"`
- `allure_report_generator_tests_dir`: here you should put test results, defaulte`"{{ allure_report_generator_srv_dir }}/test-results"`
- `allure_report_generator_reports_dir`: here the daemon will store generated allure reports, default is `"{{ allure_report_generator_srv_dir }}/allure-reports"`
- `allure_report_generator_tests_dir_owner`: default `"root"`
- `allure_report_generator_tests_dir_group`: default `"root"`
- `allure_report_generator_reports_dir_owner`: default `"root"`
- `allure_report_generator_reports_dir_group`: default `"root"`

Example
-------

    - hosts: infra01
      become: true
	  vars:
        allure_reports_dir: "/opt/allure-reports/my-project"
        allure_report_generator_tests_dir_owner: "storage"
        allure_report_generator_tests_dir_group: "storage"
      tags:
        - allure
      roles:
        - role: freehck.allure_report_generator
          allure_report_generator_ct_name: "allure-reports-my-project-surefire"
          allure_report_generator_reports_dir: "{{ allure_reports_dir }}/my-project/surefire"
        - role: freehck.allure_report_generator
          allure_report_generator_ct_name: "allure-reports-my-project-failsafe"
          allure_report_generator_reports_dir: "{{ allure_reports_dir }}/my-project/failsafe"
    
    - hosts: infra01
      become: true
      vars:
        allure_domain: "allure.example.com"
        nginx_vhost_allure:
          server_name: "{{ allure_domain }}"
          listen: "443 ssl"
          filename: "{{ allure_domain }}.443.conf"
          access_log: "/var/log/nginx/{{ allure_domain }}.443.access.log"
          error_log: "/var/log/nginx/{{ allure_domain }}.443.error.log"
          root: "{{ allure_reports_dir }}"
          extra_parameters: |
            location / {
              autoindex on;
              try_files $uri $uri/ $uri/index.html;
            }
      tags:
        - nginx
      roles:
        - role: geerlingguy.nginx
    	  nginx_vhosts:
            - "{{ nginx_vhost_allure }}"


Install
-------

This role can be installed from [Ansible Galaxy](https://galaxy.ansible.com/):

`ansible-galaxy install freehck.allure_report_generator`

License
-------

MIT

Author Information
------------------

This role was written by Dmitrii Kashin aka freehck
