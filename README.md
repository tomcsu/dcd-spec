# dcd-spec

A rough POC of how declarative continuous delivery in Spinnaker could
look.

**WIP: Still working on this README**

* Concrete examples can be seen in the [examples](examples/) directory.
* The [scratch-pad](scratch-pad) directory is just rough scribblings if you're
  curious. The main concepts will wind up distilled into this readme.

# YAML syntax note

While this repo is using YAML formatting, the actual protocol will use JSON
for transport. If templates and configurations are written in YAML, they will
need to be converted to JSON prior to being sent to Orca.

YAML was chosen for this spec because it's easier to write / grok.

# key concepts

* DCD is split into Templates and Configurations. Templates can inherit
  and decorate parent templates. Configurations are concrete implementations
  of one or more Templates.
* Composability of Templates are done through modules. Configurations can
  configure a module, or entirely replace modules if they don't fit.
* Templates can use Moustache template syntax (within strings only) for
  better flow control.
* Configurations can inject new stages or groups of stages into the final
  pipeline graph with keywords `before`, `after`, `first` and `last`.


# variables

Variables have hinted types and can be used within a template and child
templates. They require a `name`, `description` and optionally a `type`
field.

The `type` field accepts `int`, `float`, `list`, `object` and `string`.
The field is only optional if the type is `string`.

```yaml
variables:
- name: regions
  description: A list of AWS regions to deploy into
  type: list
```

# modules

Modules can be referenced by template they're defined in, each other or
replaced by child templates and the configuration. At minimum, a module
must have an `id`, `usage` and `definition`.

Modules, combined with Moustache templating can be powerful for looping
over similar template blocks, as well as swapping cloud provider functionality
from a common, standard template.

* `id` is used for referencing across templates. If child templates also 
  define a module for the same `id`, the child template's module will 
  take precedence.
* `usage` is templating-only usage documentation. It is a required field.
* `definition` is the templating value. It can be any data type and its
  value will be injected wherever the module is called.
* `variables` *(optional)* are used to define any variables that the
  module needs injected into it.

```yaml
# Modules MUST be separate documents in YAML.
---
id: deployClusterAws
usage: Defines a deploy stage cluster using the AWS cloud provider
definition:
  provider: aws
  account: mgmt
  region: "{{ region }}
```
