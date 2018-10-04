# Git

## ローカルの不要なブランチを消去するシェルスクリプト
`git branch -D` で消すこともできるが、面倒。 `brd` でインタラクティブに削除できるようにした。  

```bash
brd() {
    git branch | while read line
    do
        if [[ $line =~ \* ]] ; then
            echo -e "$line : current branch\n\033[0;36m=> skip.\033[0;39m\n"
            continue
        fi
        echo -e -n "\033[0;33m$line\033[0;39m ?(yes/no) : "
        read -r ANSWER < /dev/tty
        if [ "$ANSWER" == "yes" ] || [ "$ANSWER" == "y" ] ; then
            git branch -D $line
            echo -e "\033[0;36mdelete success.\033[0;39m\n"
        elif [ "$ANSWER" == "no" ] || [ "$ANSWER" == "n" ] ; then
            echo -e "\033[0;31mnot delete.\033[0;39m\n"
        else
            echo -e "\033[0;31mdelete failure.\033[0;39m\n"
        fi
    done
}
```
