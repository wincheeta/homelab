
# setup ssh key

generate ssh key for device
``` bash
ssh-keygen -t ed25519 -C "winchesterhj@gmail.com"

# view ssh keys
ls -al ~/.ssh
```
dont enter a file for the defaule ~/ssh dir

add the key to the ssh agent
``` bash
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519

cat ~/.ssh/id_ed25519.pub
```

clone the desired repo
``` bash
git clone git@github.com:wincheeta/coursework.git
```

add ssh key to profile/settings/ ssh and gpg keys for access to all repos
