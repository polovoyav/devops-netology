1)
sysadmin@AndrewPC:~/devops-netology/02-git-04-tools/terraform$ git show --pretty=oneline --no-patch aefea
aefead2207ef7e2aa5dc81a34aedf0cad4c32545 Update CHANGELOG.md

2)
sysadmin@AndrewPC:~/devops-netology/02-git-04-tools/terraform$ git tag --points-at 85024d3
v0.12.23

3)
sysadmin@AndrewPC:~/devops-netology/02-git-04-tools/terraform$ git show b8d720
commit b8d720f8340221f2146e4e4870bf2ee0bc48f2d5
Merge: 56cd7859e 9ea88f22f
# Видим двух родителей 56cd7859e и 9ea88f22f
sysadmin@AndrewPC:~/devops-netology/02-git-04-tools/terraform$ git rev-parse 9ea88f22f
9ea88f22fc6269854151c571162c5bcf958bee2b
sysadmin@AndrewPC:~/devops-netology/02-git-04-tools/terraform$ git rev-parse 56cd7859e
56cd7859e05c36c06b56d013b55a252d0bb7e158

4)
sysadmin@AndrewPC:~/devops-netology/02-git-04-tools/terraform$ git show --pretty=oneline --no-patch v0.12.23..v0.12.24
33ff1c03bb960b332be3af2e333462dde88b279e (tag: v0.12.24) v0.12.24
b14b74c4939dcab573326f4e3ee2a62e23e12f89 [Website] vmc provider links
3f235065b9347a758efadc92295b540ee0a5e26e Update CHANGELOG.md
6ae64e247b332925b872447e9ce869657281c2bf registry: Fix panic when server is unreachable
5c619ca1baf2e21a155fcdb4c264cc9e24a2a353 website: Remove links to the getting started guide's old location
06275647e2b53d97d4f0a19a0fec11f6d69820b5 Update CHANGELOG.md
d5f9411f5108260320064349b757f55c09bc4b80 command: Fix bug when using terraform login on Windows
4b6d06cc5dcb78af637bbb19c198faff37a066ed Update CHANGELOG.md
dd01a35078f040ca984cdd349f18d0b67e486c35 Update CHANGELOG.md
225466bc3e5f35baa5d07197bbc079345b77525e Cleanup after v0.12.23 release

5)
sysadmin@AndrewPC:~/devops-netology/02-git-04-tools/terraform$ git log --pretty=format:"%H, %as" -S"func providerSource"
5af1e6234ab6da412fb8637393c5a17a1b293663, 2020-04-21
8c928e83589d90a031f811fae52a81be7153e82f, 2020-04-02

# Берем более ранний коммит, делаем git show, чтобы убедиться, что функция там есть
git show 8c928e83589d90a031f811fae52a81be7153e82f
...
+func providerSource(services *disco.Disco) getproviders.Source {
...

6)
sysadmin@AndrewPC:~/devops-netology/02-git-04-tools/terraform$ git grep -e "func globalPluginDirs("
plugins.go:func globalPluginDirs() []string {

# Видим функцию в файле plugins.go. Ищем коммиты со строкой globalPluginDirs в файле plugins.go
sysadmin@AndrewPC:~/devops-netology/02-git-04-tools/terraform$ git log -L^:globalPluginDirs:plugins.go --pretty=format:"%H, %as" --no-patch
78b12205587fe839f10d946ea3fdc06719decb05, 2020-01-13
52dbf94834cb970b510f2fba853a5b49ad9b1a46, 2017-08-09
41ab0aef7a0fe030e84018973a64135b11abcd70, 2017-08-09
66ebff90cdfaa6938f26f908c7ebad8d547fea17, 2017-05-03
8364383c359a6b738a436d1b7745ccdce178df47, 2017-04-13

7)
sysadmin@AndrewPC:~/devops-netology/02-git-04-tools/terraform$ git log -S"func synchronizedWriters(" --pretty=format:"%H %an %as"
bdfea50cc85161dea41be0fe3381fd98731ff786 James Bardin 2020-11-30
5ac311e2a91e381e2f52234668b49ba670aa0fe5 Martin Atkins 2017-05-03

# Видим два коммита, просматриваем их git show.
sysadmin@AndrewPC:~/devops-netology/02-git-04-tools/terraform$ git show bdfea50cc85161dea41be0fe3381fd98731ff786 | grep "func synchronizedWriters("
-func synchronizedWriters(targets ...io.Writer) []io.Writer {
sysadmin@AndrewPC:~/devops-netology/02-git-04-tools/terraform$ git show 5ac311e2a91e381e2f52234668b49ba670aa0fe5 | grep "func synchronizedWriters("
+func synchronizedWriters(targets ...io.Writer) []io.Writer {

# Функция добавлен в более раннем коммите. Автор Martin Atkins.
