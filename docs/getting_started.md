# Getting Started

Welcome to the world of drive-by-wire. With code and microcontrollers, you can
reproducibly *write* machines that perform arbitrarily complex sets of tasks
on physical systems.

Cooper has a suite of software and hardware through we which we make this
happen. Originally, it was designed for Cooper IGVC, and is now open and
available for everyone to use.

## Computer Prep
You need a computer running Linux or macOS (and if you're running macOS, you
probably still need a Linux VM).

If you're running macOS:

1. Install homebrew.
2. Install virtual machine software, like Parallels (paid) or VMWare Fusion (free
  for students).

## Dependencies
### Code editor
We recommend VS Code! The GitHub Themes extension is good.

### Python and SCons
1. **Python and pip**

    If you're running macOS:

    ```
    ❯ brew install python3
    ```

    If you're running Linux, your distro probably already comes with a recent
    version of Python 3, but look up more specific instructions for your distro
    to make sure. Also ensure you have pip.

2. **SCons**

    ```
    ❯ pip3 install scons
    ```

### `direnv`
On macOS:
```
❯ brew install direnv
```

On Linux, follow distro-specific instructions online.

Then, [follow the instructions](https://direnv.net/docs/hook.html) to hook
`direnv` into your shell.

### Optional: cyme
Cyme is a modern `lsusb` replacement. If you're on macOS:
```
❯ brew install tuna-f1sh/taps/cyme
```

If you're on Linux, you can run `cargo install cyme` if you have Rust installed.

## Build Some Firmware
For now, we'll use the Cooper Motorsports repo as the example repository.

### Clone the repository
[Set up your ssh
keys](https://docs.github.com/en/authentication/connecting-to-github-with-ssh)
and:

```
❯ git clone git@github.com:CooperUnion/motorsports.git
```

Great, you have the code!

### Build something
Cool, now let's build something.

Go ahead and:
```
❯ cd motorsports
```

You should see something like:
```
❯ cd motorsports
direnv: error /Users/dmezh/motorsports/.envrc is blocked. Run `direnv allow` to approve its content
```

Say:
```
❯ direnv allow
```

Now, you can run:
```
❯ fwpio run -e shutdown
```

And that will build the firmware for the `shutdown` Motorsports ECU 🫡

Remaining dependencies will be automatically managed by the build process.
