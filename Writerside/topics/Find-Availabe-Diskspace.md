# Find Available Disk Space

## Identify Which Folders Use the Most Space

<p>When getting a warning that you're running out of disk space in Linux, and thus you need to identify files to 
remove, use the following command at the root directory, to see holistically, which folder is consuming the
most space.</p>
<p>To identify folders by how much disk space they're using:</p>
```bash
du -h --max-depth=1
```

You might need to drill down further into the folder structure to identify the files that are consuming the most space.
A common place to look would be the `/var/log` folder. To identify the files that are consuming the most space, use the
following command:

```bash
du -h --max-depth=1 /var/log
```
