## Composer requires
This requires must be saved as file `composer.json` in your project root directory. After that you can use command `composer install`.

Packages for load:
* `lx/lx-core` - core of lx-platform
* `lx/lx-doc` - documentation for classes of packages used in platform
* `lx/lx-demo` - set of demonstaration for different platform instruments
* `lx/lx-dev-wizard` - set of usefull development instruments
* `lx/lx-tools` - set of commonly used plugins and snippets in applications

```json
{
    "require":{
        "lx/lx-core":"dev-master",
        "lx/lx-doc":"dev-master",
        "lx/lx-demo":"dev-master",
        "lx/lx-dev-wizard":"dev-master",
        "lx/lx-tools":"dev-master"
    },
    "repositories":[
        {
            "type":"git",
            "url":"https://github.com/epicoon/lx-core"
        },
        {
            "type":"git",
            "url":"https://github.com/epicoon/lx-doc"
        },
        {
            "type":"git",
            "url":"https://github.com/epicoon/lx-demo"
        },
        {
            "type":"git",
            "url":"https://github.com/epicoon/lx-dev-wizard"
        },
        {
            "type":"git",
            "url":"https://github.com/epicoon/lx-tools"
        }
     ]
}
```
