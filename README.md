Red Alert Web
=============
Introduction
------------
Red Alert is an open source, browser based cluster monitoring system. Red Alert Web provides users to configue and operate backends, like shielding policies or specific machines. On browser you can grasp backends status and infomation as well.

Requirements
------------
* Red Alert Backend
* Red Alert Web package
* Data Source like [Graphite](https://graphite.readthedocs.io/en/latest/)

Installation
------------
* Get Code: `git clone http://gitlab.alibaba-inc.com/red_alert/red_alert_web.git`
* deploy
* Run on Apache Server or UWSGI Server or local script.

***common configuation***   
Red Alert Web could be running in three ways, however no matter which way you choice, *conf/red_alert_web.conf* and *static/ra_conf.js* are the configuation files must be modified. *conf/red_alert_web.conf* looks like

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

In `DEFAULT` section, `projectRoot` is red alert web's home, to fill with an real absolute path. `workspace` could be specified to any path with permission.   

As long as `DEFAULT`section configued, `Parameter` section will be generated automatically.   

- `RedAlertWebWorkRoot` is Red Alert Web's workspace home.   

- `RedAlertWebConfDir` is Red Alert Web's `sqlite` file directory.   

- `RedAlertWebTablePath` is `sqlite`'s absolte path. This sqlite db file contains `Policy`, `RedAlert`, `DataSource`, `Pairs`, `Shield` tables, that will be translated to `json` in backend.   

- `RedAlertWebAuxPath` is `raweb.aux.db`'s absolte path that is a sqlite database file as well, but only be used by web self. `raweb.aux.db` records `sqlite` table's modified status.   

- `RedAlertWebJsonPath` is `raweb.json`'s absolte path. This json will be used to verify input string.   

- `RedAlertWebVersionDir` is web's version directory that records `sqlite`'s version changes. If any table changes happened and be deployed, web will update `sqlite` file and generate a new version in *RedAlertWebVersionDir*, then the latest `sqlite` file will be placed into this version. so *RedAlertWebVersionDir* is essential.   

- `runPort` is web's listening port, that only be neccessory in local script running. If deployed on Apache Web server or run with uwsgi server, port would be configed by web servers, just ignoring this option.   

- `MaxRaBackEnd` is the max quantity of red alert backend permitted.   

- `RedAlertStoragePath` is the remote filesystem storage path. In this verison Red Alert supports local filesystem and pangu distributed filesystem now, please prefix with ```file://``` or ```pangu://``` and then make sure this directory could be accessed. If pangu has been installed, `fsUtil` is panggu's binary path or script, it's neccessary to specify while pangu used.

***Static Confiuation***   
Web server will find static files in ```static/``` directory. Modify ```ra_conf.js``` file. ```api_url``` should be specifid to your ```http://<ip>:<port>``` address or url. It looks like   

    var ra_conf = {     
        "api_url": "http://100.82.23.31:5011",   
        "current_url": "http://0.0.0.0:5011/index.html",   
        "buc_sso_url": "http://search-tools.yisou.com/buc_sso/index.php",   
        "api_timeout": 3000,   
        "admin_timeout": 10000,   
        "disable_account": true   
        };

*NOTE:* If you don't need authentication, please set `disable_account` option true. If `false` setted, `buc_sso_url` will work to verify identidy.
  
**Running**   
OK now, All basical configuations have been done, running web by `python raweb/main.py`, and then point your browser at `http://localhost:50007/index.html`. 

If you prefer to UWSGI server, we provide [uwsgi1.4](https://uwsgi-docs.readthedocs.io/en/latest/WSGIquickstart.html), just enter `uwsgi` and "making" it. Then if provided default `raweb.ini` file used, you should modify at least `raweb_install_prefix` and `http` options. The default `raweb.ini` looks like

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

Creating your own `*.ini` conf file as well, [Quickstart for Python Application](https://uwsgi-docs.readthedocs.io/en/latest/WSGIquickstart.html). If all those configuations have been done, running `./uwsgi/uwsgi --ini raweb.ini` to start web service. 

Red Alert Web could alse deployed on [Apache Server](https://httpd.apache.org/docs/2.2/).
We provide a template including a virtual host to run on `conf/` dir for reference. `raweb.conf.template` looks like

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

If you create your own `.conf` file, place it on `/etc/httpd/conf.d`, then restart your httpd service. `sudo service httpd restart`

## Quick Start
You're up and running! Red alert web is now running on port 50007.so point your browser at http://localhost:50007/index.html.

The first screen you arrived at is Policy, modify policies here. If web is the first run, `data source` in Miscellaneous Configuration should be added at least one, without data source backends can't fetch metric trees. At present red alert backend supports `Graphite` and `Amonitor`, but you can access other data source as well, just implement `fetchMetrics` interface. Then run backends, online backend's infomation and status will be showed on `console` screen as well as `sqlite` version list and `storagePath`. In this screen, please pay attention to the *Explanation*.  
 
- *Refresh Status* : Refresh and read web's current status immediately.   

- *deploy* : Any modifications saved in `aux.raweb.db` will not work, unless you deploy. Deploy will create a new version that web and backends work with.   

- *reload current* : If backends work with multiple versions of sqlite, or other accident happened, reload the latest version. All backends will receive the latest version path and then reload this version.   

- *recover* : Clean `raweb.aux.db`. If recovered, all tables in `raweb.aux.db` will be cleaned, that means all changes without deploy will disappear.   

- *reload* : If you want to rollback to old version, points to *reload*. Reload option will rollback all tables except `redAlert`, course outdate backend infomation is uesless and dangerous.

## Contribute
