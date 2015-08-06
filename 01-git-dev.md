# Git Deployment
```
cd /var
mkdir repo && cd repo
mkdir site.git && cd site.git
apt-get install -y git
git init --bare
```
`--bare` significa que el folder no va a tener el source code solo el version control.

Git tiene define 3 posibles tipos de `hooks`. `pre-receive`, `post-receive` y `update`.

`Pre-receive` se ejecutan tan pronto el servidor recibe un `push`, `update` es similar pero se para cada branch. `post-receive` se ejecuta cuando el `push` termino por completo. Este es el que nos interesa.

```
cd hooks
vim post-receive

#!/bin/sh
git --work-tree=/var/www/live --git-dir=/var/repo/site.git checkout -f
```

Ajustar permisos
`chmod +x post-receive`

### Ahora en la maquina local
```
mkdir git-deploy && cd git-deploy
git remote add live ssh://root@104.236.134.77/var/repo/site.git
vim index.html
Hello World i'm live!
git add .
git commit -m "genesis commit"
git push live master
```

## Ya hiciste un deploy a un VPS

Ahora queremos hacer deployment al servidor en un ambiente de prueba

```
cd /var/repo
mkdir beta.git && cd beta.git
git init --bare
```

```
cd hooks
vim post-receive

#!/bin/sh
git --work-tree=/var/www/beta --git-dir=/var/repo/beta.git checkout -f
```

```chmod +x post-receive```

### Ahora en la maquina local
```
git remote add beta ssh://root@162.243.153.72/var/repo/beta.git
# hacemos un cambio
git add .
git commit -m "Nueva version"
git push beta master
```
Si todo funciona bien ya hacemos deploy a live
```
git push live master
```
