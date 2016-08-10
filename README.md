Red Alert Web
=============
Introduction
------------
Red Alert is an open source, browser based cluster monitoring system. Users could configure red alert backend on\
 browser, like shielding policies and configure parameters, as well as grasp backends status and infomation.

Requirements
------------
* Red Alert Backend
* Red Alert Web package
* Data Source like [Graphite](https://graphite.readthedocs.io/en/latest/)

Installation
------------
* Get Code: `git clone http://gitlab.alibaba-inc.com/red_alert/red_alert_web.git`

Configuation
------------

***common configuation***

Red Alert Web could be started in three ways(apache, uwsgi and location). However no matter which way, *conf/red\
_alert_web.conf* and *static/ra_conf.js* are the configuation files that must be modified. *conf/red_alert_web.c\
onf* looks like

    [DEFAULT]    
    projectRoot = /var/www/html/red_alert_web    
    workspace = /var/www/html/red_alert_web    
    
    [PathConfig]   
    RedAlertWebWorkRoot = %(workspace)s/work   
    RedAlertWebConfDir = %(RedAlertWebWorkRoot)s/current   
    RedAlertWebTablePath = %(RedAlertWebConfDir)s/sqlite   
    RedAlertWebAuxPath = %(RedAlertWebWorkRoot)s/raweb.aux.db   
    RedAlertWebJsonPath = %(RedAlertWebWorkRoot)s/raweb.json   
    RedAlertWebVersionDir = %(RedAlertWebWorkRoot)s/tmp   
    
    [Parameter]   
    runPort = 50007   
    MaxRaBackEnd = 3   
    RedAlertStoragePath = file://var/www/html/foo   
    
    [fsLib]   
    fsUtil = %(projectRoot)s/fs_lib/bin/fs_util

In `DEFAULT` section, `projectRoot` is web's root directory and  `workspace` could be specified to any path with permission.   

As long as `DEFAULT`section configued, `Parameter` section would be generated automatically.   

- `RedAlertWebWorkRoot` is Red Alert Web's workspace home.   

- `RedAlertWebConfDir` is Red Alert Web's `sqlite` file directory.   

- `RedAlertWebTablePath` is `sqlite`'s absolte path. This sqlite db file contains `Policy`, `RedAlert`, `DataSource`, `Pairs`, `Shield` tables, that would be translated to `json` in backend.   

- `RedAlertWebAuxPath` is `raweb.aux.db`'s absolute path which is sqlite file as well, but only be used by web self. `raweb.aux.db` records `sqlite` table's modified status.   

- `RedAlertWebJsonPath` is `raweb.json`'s absolute path, which would be used to verify input string.   

- `RedAlertWebVersionDir` is web's version directory that records `sqlite`'s version update. If any table have been modified and deployed, a new `sqlite` version would be produced in *RedAlertWebVersionDir* so *RedAlertWebVersionDir* is essential.   

- `runPort` is an listening port, that only be neccessory in local script running. Deployed on Apache or run with `uwsgi`, port would be assigned in those server's configuration, just ignore this option.   

- `MaxRaBackEnd` is the max quantity of red alert backend permitted, if there are multiple backends,only one backend would provide the `MetricsTree`.   

- `RedAlertStoragePath` is the remote filesystem storage path. In this verison Red Alert supports local filesystem and pangu distributed filesystem, please prefixed to ```file://``` or ```pangu://``` and then make sure this directory could be accessed. If pangu has been installed, `fsUtil` is panggu's binary path or script, it's neccessary to specify.

***Static Confiuation***   
`static` directory provide static files to web server. Modify `ra_conf.js` file, `api_url` should be specifid to your `http://<ip>:<port>` address or url. It looks like   

    var ra_conf = {     
        "api_url": "http://100.82.23.31:5011",   
        "current_url": "http://0.0.0.0:5011/index.html",   
        "buc_sso_url": "http://search-tools.yisou.com/buc_sso/index.php",   
        "api_timeout": 3000,   
        "admin_timeout": 10000,   
        "disable_account": true   
        };

*NOTE:* If you don't need authentication, please set `disable_account` option true. As loog as `false` setted, `buc_sso_url` will work to verify identidy.
  
**Running**   
OK now, All basical configuations have been done, running project by `python raweb/main.py`, and then point your browser at `http://localhost:50007/index.html`. 

If you prefer to uwsgi server, we provide [uwsgi1.4](https://uwsgi-docs.readthedocs.io/en/latest/WSGIquickstart.html), just enter `uwsgi` dirctory and "making" it. If provided default `raweb.ini` file used, you should modify `raweb_install_prefix` and `http` options at least. The default `raweb.ini` looks like

    [uwsgi]
    raweb_install_prefix = /var/www/html/red_alert_web
    check-static = %(raweb_install_prefix)s/static
    daemonize2 = ./uwsgi.log
    http = 100.82.23.31:50007
    log-maxsize = 1024000
    wsgi-file = %(raweb_install_prefix)/raweb/raweb.wsgi
    master = true
    workers = 1
    threads = 10
    pidfile = ./uwsgi.pid

Creating your own `*.ini` conf file is aslo permitted, [Quickstart for Python Application](https://uwsgi-docs.readthedocs.io/en/latest/WSGIquickstart.html). Running `./uwsgi/uwsgi --ini raweb.ini` to start web service. 

Web service could alse be deployed on [Apache Server](https://httpd.apache.org/docs/2.2/).
We provide a template, a virtual host included, on `conf/` dir for reference. The `raweb.conf.template` is

    Listen 100.82.23.31:5011
    <VirtualHost 100.82.23.31:5011>
        ServerName www.raweb.com
                   DocumentRoot /var/www/html/red_alert_web
                   WSGIDaemonProcess red_alert_web python-path=/usr/ali/lib/python2.5/site-packages/:/var/www/html/red_alert_web
                   WSGIScriptAlias / /var/www/html/red_alert_web/raweb/raweb.wsgi process-group=red_alert_web application-group=%{GLOBAL}
                   CustomLog /var/log/httpd/raweb_access.log common
                   ErrorLog /var/log/httpd/raweb_error.log
                 <Directory /var/www/html/red_alert_web>
                              Order allow,deny
                              Allow from all
                </Director>
        </VirtualHost>

Rename `.template` file, drop *.template* suffix and place the file  on `/etc/httpd/conf.d`, then restart your httpd service.

## Quick Start
You're up and running! Red Alert Web is now running.

The first screen you arrived at is  policy configuation, where policies could be editted here.In shield scrren, policies in specified machines could be shield unitl specified time. `data source` in miscellaneous configuration should be added at least one when first run, without data source backends can't fetch metrics. At present backends support `Graphite` and `Amonitor`, but you can access other data sources as well, just implement `fetchMetrics` interface.In deploy screen, you could see tables' modifications. Online backend's infomation and status would be showed on `console` screen as well as `sqlite` version list and `storagePath`. In admin screen, please pay attention to the *Explanation*.  
 
- *Refresh Status* : Refresh and read web's current status immediately.   

- *deploy* : Any modifications saved in `aux.raweb.db` will not work, unless you deploy. Deploy will create a new version that web and backends work with.   

- *reload current* : If backends work with multiple versions of sqlite, or other accident happened, reload the latest version. All backends will receive the latest version path and then reload this version.   

- *recover* : Clean `raweb.aux.db`. If recovered, all tables in `raweb.aux.db` will be cleaned, that means all changes without deploy will disappear.   

- *reload* : If you want to rollback to old version, points to *reload*. Reload option will rollback all tables except `redAlert`, course outdate backend infomation is uesless and dangerous.

## Contribute
