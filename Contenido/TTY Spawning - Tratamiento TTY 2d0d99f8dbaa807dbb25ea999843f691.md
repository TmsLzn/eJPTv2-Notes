# TTY Spawning - Tratamiento TTY

Para crearnos una pseudo-consola donde podamos hacer *control c* y no tengamos problemas con el teclado ejecutamos lo siguiente:

```bash
script /dev/null -c bash # Una vez tengamos acceso a la máquina escribimos esto.
control z # Lo dejamos en segundo plano.
stty raw -echo; fg
xterm
export TERM=xterm
export SHELL=bash

```

---

### A continuación vamos a ver varios métodos para lanzarnos una TTY

```bash
/bin/sh -i # Spawn Interactive sh shell.
python -c 'import pty;pty.spawn("/bin/bash")' # Python TTY Shell.
python3 -c 'import pty;pty.spawn("/bin/bash")' # Python TTY Shell.
perl -e 'exec "/bin/sh";' # **Spawn Perl TTY Shell.
ruby -e 'exec "/bin/sh"' # Spawn Ruby TTY Shell.

```