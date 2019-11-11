# JupyterHub deployment in use at Université de Versailles

This is a [JupyterHub](https://jupyter.org/hub) deployment based on
Docker currently in use at [Université de
Versailles](https://jupyter.ens.uvsq.fr/).
fork from https://github.com/defeo/jupyterhub-docker

## 注記
docker-composeではjupyterhub,jupyterlab-throaway,reverse-proxyの3コンテナが立ち上がります。  
各ユーザのjupyter-{username}コンテナはdockerspownによって立ち上げられるため、上記のcomposeグループには入っていません。  
そのため`docker-compose down`ではjupyter-{username}コンテナが削除されません。  
そして最悪なことにjupyter-{username}コンテナを削除せずに`docker-compose up`しても、docekrspownは既にコンテナがすでに存在しているためspownできずに`500 internal server error`をユーザに返します。  

`docker-compose down`の後に`docker rm $(docker ps -aq)`でコンテナをすべて削除するようにしてくだい。  
また、`docker-compose up`の前に`docker ps -a`でjupyter-{username}コンテナがいたら`docker rm`で削除してから`docker-compose up`するようにしてください。  

## 最低限必要な変更項目

### /template.env

template.envのファイル名を.envに変更して、以下項目の値を設定してください。
```.env
OAUTH_CALLBACK_URL=xxxxx
OAUTH_CLIENT_ID=xxxxx
OAUTH_CLIENT_SECRET=xxxxx
```

### /jupyterhub/jupyterhub_config.py

ホワイトリストとadminユーザの設定をしてください。
adminユーザはJupyter-hub全体のセッションにアクセスが可能です。

```
c.Authenticator.whitelist = {'iranika','aki-kubota' }
c.Authenticator.admin_users = {'iranika', 'aki-kubota' }
```

### /reverse-proxy/traefik.toml

SSLの設定が必要です。
デフォルトでは/etc/certs/配下にcertFileとkeyFileを配置する必要があります。

```
      [[entryPoints.https.tls.certificates]]
      certFile = "/etc/certs/fullchain1.pem"
      keyFile = "/etc/certs/privkey1.pem"
```


## Features

- Containerized single user Jupyter servers, using
  [DockerSpawner](https://github.com/jupyterhub/dockerspawner);
- Central authentication to the University CAS server;
- User data persistence;
- HTTPS proxy.

## Learn more

This deployment is described in depth in [this blog
post](https://opendreamkit.org/2018/10/17/jupyterhub-docker/).

### Adapt to your needs

This deployment is ready to clone and roll on your own server. Read
the [blog
post](https://opendreamkit.org/2018/10/17/jupyterhub-docker/) first,
to be sure you understand the configuration.

Then, if you like, clone this repository and apply (at least) the
following changes:

- In [`.env`](.env), set the variable `HOST` to the name of the server you
  intend to host your deployment on.
- In [`reverse-proxy/traefik.toml`](reverse-proxy/traefik.toml), edit
  the paths in `certFile` and `keyFile` and point them to your own TLS
  certificates. Possibly edit the `volumes` section in the
  `reverse-proyx` service in
  [`docker-compose.yml`](docker-compose.yml).
- In
  [`jupyterhub/jupyterhub_config.py`](jupyterhub/jupyterhub_config.py),
  edit the *"Authenticator"* section according to your institution
  authentication server.  If in doubt, [read
  here](https://jupyterhub.readthedocs.io/en/stable/getting-started/authenticators-users-basics.html).

Other changes you may like to make:

- Edit [`jupyterlab/Dockerfile`](jupyterlab/Dockerfile) to include the
  software you like. Do not forget to change
  [`jupyterhub/jupyterhub_config.py`](jupyterhub/jupyterhub_config.py)
  accordingly, in particular the *"user data persistence"* section.

### Run!

Once you are ready, build and launch the application with

```
docker-compose build
docker-compose up -d
```

Read the [Docker Compose manual](https://docs.docker.com/compose/) to
learn how to manage your application.

## Acknowledgements

<img src="https://opendreamkit.org/public/logos/Flag_of_Europe.svg" height="20"> Work partially funded by the EU H2020 project
[OpenDreamKit](https://opendreamkit.org/).
