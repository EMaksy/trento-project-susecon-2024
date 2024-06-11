# Writing Checks Cheat Sheet

Let's load our workshop catalog so we can play with our own check file.
To do that, edit the trento-wanda configuration.

```bash
sudo vim /etc/trento/trento-wanda
```

And add the next line:

```bash
CATALOG_PATH=/home/trento/catalog/
```

And restart the service.

```bash
sudo systemctl restart trento-wanda
```

Now, let's start editing our check.

```bash
sudo vim /home/trento/catalog/ABCDEF.yaml
```