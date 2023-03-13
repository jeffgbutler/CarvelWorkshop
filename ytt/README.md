# YTT Overview

YTT is a YAML templating tool that forms the basis for many of the configuration functions in Tanzu. Many other tools
in the Tanzu and Carvel portfolio rely on YTT, so it is a fundamental tool in this space.

YTT operates in two main modes: templating and overlays.

1. You can create YAML templates that accept input values from a variety of sources. YTT includes a templating language based
   on Google Starlark that can be used to build logic into templates (loops, conditionals, etc.) YTT templates are part of the
   installation of many Tanzu tools. Further, you will most likely need to write YTT templates if you want to create your own
   Cartographer supply chains or if you use the kapp-controller. The template mode is the mode we will explore in this lesson.
2. YTT can also operate in "overlay" mode where it can modify (or patch) existing YAML. This is useful if you need to modify
   YAML that is not configured as a YTT template. If you want to build a kapp-controller package to install some third party tool,
   you may need to use YTT's overlay capabilities to modify an installation YAML to match your cluster configuration.

In templating mode, YTT will take one or more input YAML files, substitute placeholder variables with actual values
from a variety of sources, and output a final result. YTT is "YAML aware" which means that YTT understands the correct
structure of YAML documents and can build valid YAML composed of fragments. If you are familiar with compilers, you
can think that YTT builds an AST (abstract syntax tree) for YAML - which is far more sophisticated than simple string
substitution. This can be incredibly useful if you need to build a single template that can be reused many times.

Full details about YTT are here: https://carvel.dev/ytt/. That page also includes an online "playground" for YTT that
can be very helpful when learning more about YTT. You can access the playground directly
here: https://carvel.dev/ytt/#playground

**Important:** Open a terminal window in the "ytt" directory in this repository before proceeding:

```shell
cd ../ytt
```

## Simple Example
Here is a simple example. Suppose we have a YAML file named `namespace1.yaml` like this:

```yaml
#@ load("@ytt:data", "data")
apiVersion: v1
kind: Namespace
metadata:
  name: #@ data.values.namespace1
```

We can execute YTT with the following command:

```shell
ytt -f namespace1.yaml -v namespace1=yttns1
```

The output looks like this:

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: yttns1
```

All YTT commands look like YAML comments, so YAML templates are usually valid YAML and will pass validation tools.
In the source file, there are two YTT commands:

1. `#@ load("@ytt:data", "data")` - this instructs YTT to load that `data` module and make all the `data` values
   available with the key `data.values` (`values` is built in and provides access to the individual data elements.)
2. `#@ data.values.namespace1` - this instructs YTT to insert the data value `namespace1` into the YAML at this position

Data values can come from a variety of sources - most commonly from command line flags as shown above, or from a file
as we will show below.

## Multiple Input Files

If you supply more than one input file to YTT, the output will be a single consolidated YAML file. For example, suppose
there are two input files named `namespace1.yaml` and `namespace2.yaml` as shown below:

```yaml
#@ load("@ytt:data", "data")
apiVersion: v1
kind: Namespace
metadata:
  name: #@ data.values.namespace1
```

```yaml
#@ load("@ytt:data", "data")
apiVersion: v1
kind: Namespace
metadata:
  name: #@ data.values.namespace2
```

We can execute YTT with the following command:

```shell
ytt -f namespace1.yaml -f namespace2.yaml -v namespace1=yttns1 -v namespace2=yttns2
```

The output looks like this:

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: yttns1
---
apiVersion: v1
kind: Namespace
metadata:
  name: yttns2
```

As you can see, YTT created a single output YAML and substituted the correct values in each input file.

## Using a Values Input File

Rather than specifying data values with command line flags, you can also use a file (commonly named `values.yaml`) to
supply input values. For example, suppose we have the same two input files named `namespace1.yaml` and `namespace2.yaml` as shown below:

```yaml
#@ load("@ytt:data", "data")
apiVersion: v1
kind: Namespace
metadata:
  name: #@ data.values.namespace1
```

```yaml
#@ load("@ytt:data", "data")
apiVersion: v1
kind: Namespace
metadata:
  name: #@ data.values.namespace2
```

But now we have another file named `values.yaml` with content as shown below:

```yaml
namespace1: yttns1
namespace2: yttns2
```

We can execute YTT with the following command:

```shell
ytt -f namespace1.yaml -f namespace2.yaml --data-values-file values.yaml
```

The output looks like this:

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: yttns1
---
apiVersion: v1
kind: Namespace
metadata:
  name: yttns2
```

As you can see, YTT created the same output as before and substituted the correct values in each input file.

## YTT Schema

A schema file can be used to document the variables that are expected in a template, and provide default values for them.

Here is a simple example. Suppose we have a YAML file named `schema.yaml` like this:

```yaml
#@data/values-schema
---
namespace1: yttnamespace

#@schema/nullable
namespace2: ""
```

This file indicates two variables - our familiar `namespace1` and `namespace2`. The `namespace1` variable has a default value of
"yttnamespace". The `namespace2` variable is in a section denoted "nullable" - which means that if no value is specified for that
variable, it's value will be null.

We can try it out with this command:

```shell
ytt -f namespace1.yaml -f namespace2.yaml -f schema.yaml
```

The output is as follows:

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: yttnamespace
---
apiVersion: v1
kind: Namespace
metadata:
  name: null
```

You can see that the default values were used for both `namespace1` and `namespace2`. The "null" for `namespace2` isn't great - we'll deal
with that in the next section.

Of course, we can supply values for these variable in the normal way. So this command produces what we've seen before:

```shell
ytt -f namespace1.yaml -f namespace2.yaml -f schema.yaml --data-values-file values.yaml
```

## YTT Functions and "If"

You can define functions in YTT that do a variety of things. One very useful thing to do with a function is to calculate and return a
YAML fragment.

Suppose we have an input file named `YTTFunctions.yaml` like this:

```yaml
#@ load("@ytt:data", "data")

#@ def labels():
source: 'carvel-workshop'
generated: true
#@ end

#@ def namespace(name):
apiVersion: v1
kind: Namespace
metadata:
  name: #@ name
  #! Blank line below ignored by YTT

  labels: #@ labels()
#@ end

--- #@ namespace(data.values.namespace1)
#@ if/end data.values.namespace2:
--- #@ namespace(data.values.namespace2)
```

There's a lot to unpack here!

1. Functions begin with `def function_name(parameters...):` and end with `end`. On line 3 we define a function named `labels`
   that accepts no parameters. This function returns a YAML fragment with two nodes - two hard coded labels.
2. On line 8 we define a function named `namespace` that accepts one parameter - called `name`. In this case, the function is
   returning a definition of a Kubernetes namespace that is templated - we can pass a `name` to the function and it will stamp
   out a namespace.
3. On line 12 we are using the parameter passed into the function
4. On line 13 we have a YTT comment (starts with `#!`) YTT will strip comments in the resulting YAML.
5. On line 15 we are calling the `labels` function. YTT is smart enough to know that the YAML fragment returned from the `labels`
   function should be a child of the node on line 15. We don't need to worry about indenting and formatting - YTT will do it for us
6. On line 18 we call the `namespace` function passing in an expected input value. Again, the resulting YAML will be properly
   indented and formatted as a child of the node on line 18.
7. On line 19 there is an "if" statement using the shortcut `if/end`. We can use a shortcut like this when there is only one line
   of yaml to be written with the `if`. If we need more than one thing
   output from the "if" statement we can move the "end" to a separate line as with functions. This "if" statement checks for a null value
   (which is our default for `namespace2`) - it will only write the YAML fragment if the value is not null.

We can execute YTT with the following command:

```shell
ytt -f YTTFunctions.yaml -f schema.yaml --data-values-file values.yaml
```

The output will look like this:

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: yttns1
  labels:
    source: carvel-workshop
    generated: true
---
apiVersion: v1
kind: Namespace
metadata:
  name: yttns2
  labels:
    source: carvel-workshop
    generated: true
```

Is that what you expected? What happens if we omit the values file?

```shell
ytt -f YTTFunctions.yaml -f schema.yaml
```

Now we just get a single namespace with the default value:

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: yttnamespace
  labels:
    source: carvel-workshop
    generated: true
```

## YTT Loops

Now suppose that we want to produce a non-determinate number of namespace. For this we can use a loop. We'll change the
template to look like this:

```yaml
#@ load("@ytt:data", "data")

#@ def labels():
source: 'carvel-workshop'
generated: true
#@ end

#@ def namespace(name):
apiVersion: v1
kind: Namespace
metadata:
  name: #@ name
  #! Blank line below ignored by YTT

  labels: #@ labels()
#@ end

#@ for/end ns in data.values.namespaces:
--- #@ namespace(ns)
```

The template is the same except for the end where we specify a for loop. We also change the input file to look like this:

```yaml
namespaces:
- yttns1
- yttns2
- yttns3
```

We can execute YTT with the following command:

```shell
ytt -f YTTLoops.yaml --data-values-file values-namespaces.yaml
```

The output will look like this:

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: yttns1
  labels:
    source: carvel-workshop
    generated: true
---
apiVersion: v1
kind: Namespace
metadata:
  name: yttns2
  labels:
    source: carvel-workshop
    generated: true
---
apiVersion: v1
kind: Namespace
metadata:
  name: yttns3
  labels:
    source: carvel-workshop
    generated: true
```


## YTT Built-In Functions

YTT comes with many built-in functions for building more complex YAML. We will show one more example using the `json` and `base64`
modules to create a Kubernetes secret.

Suppose we have an input file named `Secret.yaml` like this:

```yaml
#@ load("@ytt:data", "data")
#@ load("@ytt:json", "json")
#@ load("@ytt:base64", "base64")
apiVersion: v1
kind: Secret
metadata:
  name: #@ data.values.registry.secret.name
type: kubernetes.io/dockerconfigjson
data:
  #@ registry_creds = {"username": data.values.registry.username, "password": data.values.registry.password}
  #@ encoded_creds = base64.encode(json.encode({"auths": {data.values.registry.server: registry_creds}}))
  .dockerconfigjson: #@ encoded_creds
```

This file loads the normal `data` module we are used to, but also loads `json` and `base64` modules. These modules
include encoding functions we can use to encode JSON (in case a supplied password has any special characters), and
then we can Base64 encode a string for setting the secret value. You can also see that we declare local variables
names `registry_creds` and `encoded_creds` using YTT.

Now we can combine this file with a `SecretValues.yaml` file like this:


```yaml
registry:
  server: docker.io
  username: fred
  password: password1234
  secret:
    name: my-secret
```

We can execute YTT with the following command:

```shell
ytt -f Secret.yaml --data-values-file SecretValues.yaml
```

The output looks like this:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: my-secret
type: kubernetes.io/dockerconfigjson
data:
  .dockerconfigjson: eyJhdXRocyI6eyJkb2NrZXIuaW8iOnsicGFzc3dvcmQiOiJwYXNzd29yZDEyMzQiLCJ1c2VybmFtZSI6ImZyZWQifX19
```

You can see that YTT has composed a properly encoded secret for the Kubernetes secret definition.

Normally, YTT does not save its output anywhere - it just pipes the output to stdout.
You can pipe YTT output to `kubectl` and create the secret in a single command:

```shell
ytt -f Secret.yaml --data-values-file SecretValues.yaml | kubectl create -f-
```

Many of the Tanzu configuration commands make use of YTT to perform functions like this. In addition, the Kapp Controller and
Cartographer both support YTT templating in their CRDs.

[Next (Kbld Overview) -&gt;](../kbld/README.md)
