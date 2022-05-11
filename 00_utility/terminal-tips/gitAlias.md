# Git alias

Sono qui elencati dei comodi alias per i comandi git più comuni, se li si vuole aggiungere si consiglia di metterli nel `.bashrc` o dentro `~/.bash_aliases`(in caso da creare)

```bash
###################################################
#################### Git Alias ####################
###################################################

# git display information
alias gg="git gui &"
alias gk="gitk --all &"

# git terminal information 
alias gs="git status"
alias gitlog='git log --all --decorate --graph'

# git update
alias gu="git pull; git submodule update"
alias gf="git fetch"

# Display git Help command
function gh 
{
    echo "#### Reset Operation ####"
    echo "> Cancella ultima commit locale, eliminando le modifiche"
    echo '  git reset --hard HEAD~1'    
    echo "> Cancella ultima commit locale, mantenendo le modifiche"
    echo '  git reset --soft HEAD~1'

    echo ""
    echo "#### Branch Operation ####"
    echo "> Creare branch"
    echo '  git checkout -b nome'
    echo '  git push -u origin nome'
    echo "> Creare tag"
    echo '  git tag -n'
    echo '  git tag -a -m "commento" v0.0.x'
    echo "> Push del singolo tag"
    echo '  git push origin v1.3.2'
    echo "> Push di tutti i tag"
    echo '  git push --tags'

    echo ""
    echo "#### Local Repository clean ####"
    echo "> Cosa cancella"
    echo '  git clean -xdn'
    echo "> Cancella"
    echo '  git clean -xdf'

    echo ""
    echo "#### Sub module operation ####"
    echo "> Aggiungere un sotto modulo"
    echo '  git submodule add <subModule-path>'
    echo "> Aggiornare il Sotto-modulo all'attuale livello del pull"
    echo '  git submodule update'

    echo ""
    echo "#### Local Branch Clean Operation ####"
    echo "> Remove all your local git branches but keep dev"
    echo '  git branch \| grep -v "dev" \| xargs git branch -D'
    echo "> Remove all your local git branches but keep current"
    echo '  git branch \| grep -v ^* \| xargs git branch -D'
}
```

## ungit

Come ulteriore strumento, grafico, per operare su git, si suggerisce l'uso di `ungit`, progetto open-source, di cui i binari sono presenti in questo repository [qui](../01_Utility-Bin/ungit/README.md).

---

[GO ---> BACK](README.md)
