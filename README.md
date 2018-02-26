creativeshop magento patch set
==============================

## How to apply patches in your *creativeshop* project?

Add to your `composer.json`:
```json
{
    "require": {
        "cweagans/composer-patches": "~1.0",
        "creativestyle/magento-patches": "dev-develop"
    },
    "extra": {
        "patches-file": "composer.patches.json",
        "enable-patching": true,
    }
}
```

_Tip: Use `develop` branch for magento 2.1 and `develop-m2.2` for magento 2.2`._

Then create `composer.patches.json` file containing:

```json
{
    "patches": {}
}
```

This setup will install only the essential patches.
In order to apply chosen optional patches find the patch you want
in `optional.patches.json` in this repository and copy-paste
the entry to your project's `composer.patches.json` file.

## Gotacha's

In order for the patching to work you need the `patch` command.

In CentOS just do `yum install patch`. It should be available
out-of-the-box on MacOS.

### Double build

*The patch repository cannot be updated and patches applied on the same
run.*

First run will update/install the patches, but the patching will fail
because they will not exist yet. Subsequent build will apply them.
There is no good way to fix this besides storing patches remotely
and fetch them using HTTP. We can't do that, because we want to store
them here.

This means that when you add something to the patch repository
or add the patch packge to your project the patches will be applied
only at the subsequent build!

_This has one big downside - the patching will fail on first run -
and this is to be expected - so we cannot enable the option which
will fail the whole composer run on patch fail._

## How to create a patch?

_Unfortunately I haven't found a way to install magento modules
from source using composer. It seems magento repo contains only
dist versions._

This means that for now you have to install module you want to patch
by other means.

Let's say we'll patch the `customer` module.

Go into your creativeshop project directory, then:

```bash
# Go into the module directory
cd vendor/magento/module-customer
# Initialize git repository
git init
# Create a dummy commit to save current state
git add . && git commit -m "dummy"
```

Now make your changes to the module, test them create the patch.
```
git add some/files
git diff --cached > your-descriptive-name.patch
```

Congratulations! The file `your-descriptive-name.patch` contains
a fresh patch.

Now copy the patch file to the `magento-patches` project.
If the patch fixes basic magento bugs and does not introduce any
breaking changes put it into `essential` patches, otherwise, use
`optional`.

If the patch is essential add an entry to the respective `composer.json`
file in `extra` section:
```
{
    "extra": {
        "patches": {
            "magento/module-customer": {
                "Short fix description": "vendor/creativestyle/magento-patches/essential/your-descriptive-name.patch"
            }
        }
    }
}
```

If the patch is optional add this entry to `optional.patches.json`
for future reference. This fill will not be used directly but will
come handy for copy-pasting patch definitions.

### Bonus step - please do that!

Edit the patch file with a description. Put it before the header -
the line starting with `---`. Following information may be important:
 - What is the rationale behind the patch?
 - Does it solve a certain issue - link to github.
 - Is it an external patch? Provide original link.
 - Can it break something?
 - Do you know in which version the official fix will be released?

See `essential/customer-grid-indexing-fix.patch` for an example.

