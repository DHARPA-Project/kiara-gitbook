# topic modeling in Jupyter notebooks

## Set up kiara

Before we begin, we must ensure that Kiara and its plugins are correctly installed.

To set up kiara, paste and run the following code:

```
try:
    from kiara_plugin.jupyter import ensure_kiara_plugins
except:
    import sys
    print("Installing 'kiara_plugin.jupyter'...")
    !{sys.executable} -m pip install -q kiara_plugin.jupyter
    from kiara_plugin.jupyter import ensure_kiara_plugins

ensure_kiara_plugins()

from kiara import KiaraAPI
kiara = KiaraAPI.instance()
```

This will check (and if needed, install) everything required to use Kiara inside your notebook.

## Chooce a text corpus

Now that the environment is ready, we need a small collection of texts to work with.

For this tutorial, we use:

* La Rassegna, an Italian-language newspaper from the early 20th century
* Sampled from the open-access collection ChroniclItaly 3.0  (Viola and Fiscarelli, 2021; Viola, 2021)
* Files contain OCRed front pages, with filenames including metadata (like publication date)

