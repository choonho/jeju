# Developer Environment

# Packages
For developer, many useful packages exist.

~~~bash
apt-get -y install vim
apt-get -y install cscope
apt-get -y install ctags
apt-get -y install git
~~~

# Update .vimrc

edit .vimrc
~~~text
set expandtab
set tabstop=4
set shiftwidth=4
set visualbell
colorscheme koehler
syntax on
highlight Comment ctermfg=green


if filereadable("./cscope.out")
cs add cscope.out
endif
~~~

# Update mkcs

This is cscope environment

edit mkcs
~~~text
#!/bin/bash
CSFILE=cscope.files

if [ -f cscope.files ]; then
        rm -rf cscope.*
fi

if [ -f tags ]; then
        rm tags
fi

while [ $# -ne 0 ]
          do
              echo "Current Directory: $1 , Remaining No. of directories: $#"
              #Pass $1 to some bash function or do whatever
                  find $1 \( -name '*.c' -o -name '*.cpp' -o -name '*.cc' -o -name '*.h' -o -name '*.s' -o -name '*.S' \) -print >> cscope.files
              shift
done

ctags -L $CSFILE
cscope -i $CSFILE
~~~

Change file permission to 755

~~~bash
chmod 755 mkcs
git config --global user.name "Choonho Son"
git config --global user.email choonho.son@gmail.com
git config --global core.editor vim
~~~
