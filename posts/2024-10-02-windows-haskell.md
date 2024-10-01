---
title: 'Haskell ❤️ Windows'
author: Javier
---

This document describes my setup for working with Haskell on Windows. Notice I
don't know much of the intricacies of Windows but it seems I was able to make it
work.

------------------------------------------------------------------------------------------------

# Environment

## Git

Install git using `scoop` in PowerShell:

```pwsh
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
Invoke-RestMethod -Uri https://get.scoop.sh | Invoke-Expression
scoop install git
```

Notice we are installing Git for Windows with this. It seems like it has worked
properly for me until now. Some configs:

```
➜ cat .gitconfig
[user]
  name = ...
  email = ...
[core]
  autocrlf = input
  sshCommand = C:/Windows/System32/OpenSSH/ssh.exe
[gpg]
  program = C:/Program Files (x86)/GnuPG/bin/gpg.exe
[core]
  symlinks = true
 ```

For `gpg`, I use [`Gpg4win`](https://www.gpg4win.org/download.html), and for
`ssh` I use [Window's
OpenSSH](https://learn.microsoft.com/en-us/windows-server/administration/openssh/openssh_install_firstuse)

## MSYS2

The developer environment will be based on
[MSYS2](https://www.msys2.org/). Download the installer from their webpage and
follow the wizard steps.

### System upgrade

Run `pacman -Syuu` twice, to update the runtime and to update the packages.

### Setting the $HOME for MSYS2

Add this line to your `/etc/nsswitch.conf` file from inside MSYS2.
```
db_home: <whatever-was-here> windows
```
and restart the terminal.

### Choosing an environment

GHC ships a minimal toolchain in order to not depend on the user's toolchain
(currently 0.8 [here](https://downloads.haskell.org/ghc/mingw/), check
[here](https://gitlab.haskell.org/ghc/ghc/-/blob/master/mk/get-win32-tarballs.py#L11)
for the version on your particular GHC).

Before GHC 9.4, the base environment was GCC-based, thus the best way to align
things was to use a `MINGW64` environment.

After GHC 9.4, the base environment is clang-based. So I would suggest using the
`CLANG64` environment.

### Inheriting the Windows' PATH

Add the argument `-use-full-path` to the `msys2_shell.cmd` (right-click on the
icon that appears when you search for MSYS2 in your start menu, and modify the
shortcut).

### Essential packages

Some packages that I find very useful to have around:

- `mingw-w64-clang-x86_64-pkgconf`: there is a `msys2/pkgconf` package but it
  doesn't understand Windows paths in `PKG_CONFIG_PATH`... Sadly this means that
  if you change environments you will have to switch the `pkg-config` package
  (or install a new one).
- `base-devel`: generally useful
- `mingw-w64-clang-x86_64-zlib`
- `mingw-w64-clang-x86_64-openssl`

or in one line:
```bash
pacman --noconfirm -S base-devel mingw-w64-clang-x86_64-pkgconf mingw-w64-clang-x86_64-zlib mingw-w64-clang-x86_64-fd mingw-w64-clang-x86_64-jq mingw-w64-clang-x86_64-emacs mingw-w64-clang-x86_64-python mingw-w64-clang-x86_64-python-pip
```

## Windows Terminal

I would suggest installing the [Windows
Terminal](https://www.microsoft.com/store/productId/9N0DX20HK701?ocid=pdpshare)
and [`FiraCode Nerd font`](https://www.nerdfonts.com/font-downloads) (make sure
to install it for all users or Windows Terminal will complain).  There you can
configure the different profiles you want to use. My settings are as follows
(notice the `-use-full-path` and the `-shell zsh`):

```json
"defaultProfile": "<some profile guid from below>",
"profiles":
 {
   "defaults": {},
   "list":
   [
      {
          "commandline": "C:\\msys64\\msys2_shell.cmd -defterm -here -no-start -clang64 -shell zsh -use-full-path",
          ...
          "name": "MSYS2 / CLANG64"
      },
      {
          "commandline": "C:\\Program Files\\PowerShell\\7\\pwsh.exe",
          "name": "Windows PowerShell"
      }
  ]
}
```

## PowerShell

Install the latest PowerShell with `winget`:
```pwsh
winget install --id Microsoft.WindowsTerminal --source winget
```

Point your Windows Terminal profile to `C:\Program Files\PowerShell\7\pwsh.exe`.

Add this to your profile
(`~/Documents/PowerShell/Microsoft.PowerShell_profile.ps1`) to enable movement
like emacs on the command line:

```pwsh
Import-Module PSReadLine
Set-PSReadLineOption -EditMode Emacs
```

----------------------------------------------------------------------------------------------

# Haskell

## GHCup

Set the following user variables:

| Var                         | Value                                                              |
|-----------------------------|--------------------------------------------------------------------|
| `GHCUP_INSTALL_BASE_PREFIX` | `C:\`                                                              |
| `GHCUP_MSYS2`               | `C:\msys64`                                                        |
| `Path`                      | Add `C:\ghcup\bin` and `C:\Users\Javier\AppData\Roaming\cabal\bin` |

Then go to [GHCup](https://www.haskell.org/ghcup) and get the POSIX install
command:

```bash
curl --proto '=https' --tlsv1.2 -sSf https://get-ghcup.haskell.org | sh
```

And run it in an MSYS2 shell. Restart the terminal and run `ghcup tui` to select
the latest versions of `cabal`, `stack` and the desired `ghc`.

### Integrate with cabal

GHCup will have modified your cabal global config like this:
```cabal
extra-include-dirs: C:\msys64\clang64\include
extra-lib-dirs: C:\msys64\clang64\lib
extra-prog-path: C:\ghcup\bin,
                 C:\Users\<USER>\AppData\Roaming\cabal\bin,
                 C:\msys64\clang64\bin,
                 C:\msys64\usr\bin
```

This might lead to problems sometimes, see
[this](https://cabal.readthedocs.io/en/latest/how-to-run-in-windows.html#msys2-environments-and-packages).

I would say leave it if using `CLANG64` and `GHC >= 9.4`, but if using `MINGW64`
you might need to comment the include and lib dirs.

I use this function to switch environments if needed:
```bash
cabal-msys2-env () {
  sed -i "s/\(ucrt\|mingw\|clang\)/$1/g" $(cygpath -u "$(cabal --help | tail -n1 | sed 's_\r__g' | tr -d ' ')")
}
```

> NOTE: while [this](https://gitlab.haskell.org/ghc/ghc/-/issues/25031) is fixed, I suggest adding the following to your cabal config
> ```
> program-default-options
>   ghc-options: -optc-Wno-pragma-pack -optc-Wno-macro-redefined -optc-Wno-missing-declarations
> ```

## Profit!

![](https://gist.github.com/user-attachments/assets/2f72c31f-887c-4a6e-b6f7-6fec74081d47)
