# Kubernetes and YAML syntax support in VSCode
### Instal this plugin

YAML Language Support by Red Hat
Provides comprehensive YAML Language support to Visual Studio Code, via the yaml-language-server, with built-in Kubernetes syntax support.

YAML support is enabled by default, to have it K8 syntax enabled, update settings.json of the extension to add below line adn restart VSCode.
`
    "yaml.schemas": {
        "kubernetes": "*.yaml"
    }
        `