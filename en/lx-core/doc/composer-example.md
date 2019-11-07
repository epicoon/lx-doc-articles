## Composer requires
This requires must be saved as file `composer.json` in your project root directory. After that you can use command `composer install`.

Packages for load:
* `lx/lx-core` - core of lx-platform
* `lx/lx-tools` - set of commonly used plugins and snippets in applications
* `lx/lx-model` - ORM for lx-platform
* `lx/lx-auth` - authentication OAuth2 and authorisation RBAC tools
* `lx/lx-doc` - documentation for classes of packages used in platform
* `lx/lx-demo` - set of demonstaration for different platform instruments

```json
{
    "require":{
        "lx/lx-core":"dev-master",
        "lx/lx-tools":"dev-master",
        "lx/lx-model":"dev-master",
        "lx/lx-auth":"dev-master",
        "lx/lx-doc":"dev-master",
        "lx/lx-demo":"dev-master"
    },
    "repositories":[
        {
            "type":"git",
            "url":"https://github.com/epicoon/lx-core"
        },
        {
            "type":"git",
            "url":"https://github.com/epicoon/lx-tools"
        },
        {
            "type":"git",
            "url":"https://github.com/epicoon/lx-model"
        },
        {
            "type":"git",
            "url":"https://github.com/epicoon/lx-auth"
        },
        {
            "type":"git",
            "url":"https://github.com/epicoon/lx-doc"
        },
        {
            "type":"git",
            "url":"https://github.com/epicoon/lx-demo"
        }
    ],
    "scripts": {
        "post-update-cmd": [
            "php vendor/lx/lx-core/lx-install"
        ]
    }
}
```
