# kuban
Kuban is a custom kubernetes wrapper for ansible that make possible to build kubernetes reusable charts simpler


How kuban will work look like `main.yaml`:
```yaml

- name: Rabbitmq configurations files
  configMap:
    name: {{ project.name }}-config
    labels: {{ project.labels }}
    data:
      - {{ lookup('file', 'enabled_plugins') }}
      - {{ lookup('template', 'rabbitmq.conf') }}

- name: Rabbitmq secrets
  secret:
    name: {{ project.name }}-auth
    labels: {{ project.labels }}
    type: Opaque
    data: 
      user: "rabbitmq-user"
      password: {{ lookup('onepassword', 'rabbitmq-user', field='rabbitmq-password') }}

- name: Gather config files
  find:
    paths: ./init-config/
  register: config-files

- name: Rabbitmq initial config
  configMap:
    name: {{ project.name}}-init-{{ item.name }}
    labels: {{ project.labels }}
    data:
      - {{ item.name }} :|
        {{ lookup('file', item.path) }}
  loop: "{{ config-files }}"

- name: Rabbimtq Statefullset
  include: statefullset.yaml

```

and next file `statefullset.yaml' can look like:

```yaml

- name: Rabbitmq-statefullset creation
  statefullSet:
    name: {{ project.name }}-service
    labels: {{ project.labels }}
    spec: 
      containers:
        image: {{ project.image }}:{{ project.version }}
        ...

```

What difference to Helm Charts - ansible lookups, ready to use modules that can analyse code and other parameters and based on that create proper constructions. Like in example loading files from proper directory as configMaps.

How to run it:
```bash
kubanctl create ./my-code-here
```

Example structure of directory `/my-code-here`:
```
my-code-here/
  variables.yml
  main.yaml
  statefullset.yaml
  init-config/
    users.json
    auth.conf
```

Variables files can be stored out of repository to make it more flexible:
```bash
kubanctl create ./my-code-here -v variables.yaml -v /tmp/s3conf/var.yaml
```
