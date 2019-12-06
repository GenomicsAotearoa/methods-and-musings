# Preferred Editors
For general scripting and coding, most GA postdocs prefer [vscode](https://code.visualstudio.com/).

For interactive scripting and plotting, use [Jupyter Notebook](https://jupyter.org/) for Python (other language kernels available, though) and use [RStudio](https://rstudio.com/) for R.

## vscode
### Extensions
vscode supports a large array of useful extensions. Here are some recommended ones:
#### [TabNine](https://tabnine.com/) (Works with most languages)
Neural-net backed smart autocomplete for coding. Make sure to enable [semantic completion](https://tabnine.com/semantic) as well.

#### [SSH FS](https://marketplace.visualstudio.com/items?itemName=Kelvin.vscode-sshfs) (Language Agnostic)
Edit files remotely through an SSH connection.

#### [Git Lens](https://github.com/eamodio/vscode-gitlens) (Language Agnostic)
Tools for working with Git repositories.

#### [vscode-gist](https://github.com/kenhowardpdx/vscode-gist) (Language Agnostic)
Integrated tools for supporting and working with gists (short GitHub code snippets).

#### [Remote Development](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.vscode-remote-extensionpack)
An extension pack that lets you open any folder in a container, on a remote machine, or in WSL and take advantage of VS Code's full feature set. See [here](https://code.visualstudio.com/docs/remote/remote-overview) for detailed overview and setup.

#### [markdown-all-in-on](https://marketplace.visualstudio.com/items?itemName=yzhang.markdown-all-in-one)
Everything you need to work with Markdown in VScode.

#### [vscode-pandoc](https://marketplace.visualstudio.com/items?itemName=DougFinke.vscode-pandoc)
Renders Markdown via Pandoc. Allows you to render Markdown out to html, pdf and even docx.

#### [vscode-spotify](https://marketplace.visualstudio.com/items?itemName=shyykoserhiy.vscode-spotify)
Control Spotify within VScode, no need to leave your IDE anymore!

#### [code-settings-sync](https://marketplace.visualstudio.com/items?itemName=Shan.code-settings-sync)
Synchronize Settings, Snippets, Themes, File Icons, Launch, Keybindings, Workspaces and Extensions Across Multiple Machines Using GitHub Gist.

#### Language Extensions
vscode has extensions for pretty much every programming language and will often suggest installation of them when you begin editing those types of files. There are often multiple extensions for a single language, so it may be worth identifying the one you most prefer before installing a suggested one. Number of downloads, ratings, and recent updates are good metrics when deciding a specific extension.

#### [Kite](https://kite.com/) (Python-only)
Neural-net backed autocomplete for Python (DeepNine also supports Python, but you may want to check out Kite as well).

#### Other useful extensions
Simply search in the marketplace for these
* SnakeMake Language (Syntax highlighting and snippet support for snakemake files)
* Nextflow (Syntax highlighting)
* CWL (Syntax highlighting for common workflow-language documents)
* YAML (For working with YAML files)
* TOML (For working with TOML files)
* Singularity (Syntax highlighting)

# GitHub Tips
## [GitHub Desktop](https://desktop.github.com/) (Windows or Mac)
GUI interface to work with GitHub

## 2FA for GitHub (and SSH Servers)
[Krypton](https://krypt.co/) makes 2FA easy by using your phone as a second form of authentication. No need to use 6 digit codes from an authenticator app. Also works on the command line and via Chrome. You can [sign your git commits and use your phone as a 2FA ssh-key](https://krypt.co/developers/) too.

## Git LFS
[Git Large File Storage](https://git-lfs.github.com/) is a repo-specific installation for when you need to store large files in your repo above the 100Mb GitHub limit. It is best to avoid this when possible (but not always possible, such as videos, high-def media, or very large example files). You get some free storage from GitHub but will quickly need to pay a monthly fee, so best to use only when required.

# Language-specific Tips

## R
### Microsoft R Open
RStudio works with both standard R distribution and also [Microsoft R Open](https://mran.microsoft.com/open), which is an enhanced stability focused release of R with enhancements for [reproducibility](https://mran.microsoft.com/documents/rro/reproducibility) and [https://mran.microsoft.com/rro#intelmkl1](performance) by having multi-threaded capabilities built-in; useful when working with large statistical tests or genomic data.

## Python
### Anaconda
[Anaconda](https://www.anaconda.com/distribution/) is a python library/module and environment manager, but is becoming a more generalized package manager for bioinformatics software thanks to [bioconda](https://bioconda.github.io/user/install.html). 
SnakeMake and Nextflow workflow managers both support conda environments, allowing portability and reproducibility, making anaconda a must-have. It can be installed locally so you do not need root.
#### Creating a new environment
```shell
$ conda create -n py36 python=3.6 pip jupyter
```
Creates a new environment called **py36** which has python version 3.6, pip, and jupyter. To activate just issue the activate command.
```shell
$ conda activate py36
```
And to deactivate, just do
```shell
$ conda deactivate
```
#### As a bioinformatics package manager
As a bioinformatics package manager.
```shell
$ conda create -n vcf bioconda::vcftools bioconda::bcftools bioconda::samtools bioconda::bamtools
```
Creates a new environment called vcf which contains vcftools, bcftools, samtools, and bamtools. Activate and deactivate as necessary.
