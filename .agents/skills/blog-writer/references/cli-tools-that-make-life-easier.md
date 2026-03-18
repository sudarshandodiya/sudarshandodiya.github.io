## Before We Begin

**Note:** The CLI tools discussed in this post are primarily tailored for macOS and Linux users. While some of these tools may work on Windows, others might not be available or require additional setup to function properly.

Before we dive in, let’s talk about terminal emulators. Your command line tools are only as good as the terminal they run in. While the default terminal works fine for most, switching to a modern emulator can really level up your experience. Some popular options include:

- **Alacritty** (macOS, Windows, Linux)
- **Kitty** (macOS, Linux)
- **Wezterm** (macOS, Windows, Linux)
- **Ghostty** (macOS, Linux)
- **iTerm2** (macOS)
- **Terminal** (Windows: good WSL support)

I personally use **Kitty** for its rich feature set and stability. What makes it feature rich are features such as:

- **Quick access terminal** (like Quake): [Kitty Quick Access](https://sw.kovidgoyal.net/kitty/shell-integration/#ssh)
- **Cursor trail**: [Kitty Cursor Trail](https://sw.kovidgoyal.net/kitty/conf/#opt-kitty.cursor_trail)
- **Plugins** (“Kittens”): [Kitty Kittens](https://sw.kovidgoyal.net/kitty/kittens/custom/)
- **Integrations**: [Kitty Integrations](https://sw.kovidgoyal.net/kitty/shell-integration/)

If you spend a significant amount of time in your terminal, even small enhancements can greatly improve your experience.

## CLI Tools

### Shell

The second most important thing in a CLI environment is the shell you’re working with. **Bash** is the default for a reason: it’s reliable and gets the job done. But once you start exploring, you’ll find other shells that offer features you might not want to live without.

The most common alternative is **zsh**, especially when paired with the **Oh-My-Zsh** plugin manager. With this setup, you get things like Powerlevel prompts, syntax highlighting, and auto-suggestions. These might sound like small additions, but once you get used to them, it’s hard to go back.

That said, configuring Oh-My-Zsh can get a bit complicated, especially when you need to manually clone repositories for some plugins to work. If you want something simpler, **Antidote** is a solid replacement. It supports Oh-My-Zsh plugins, so you can switch over at your own pace without losing your setup.

Personally, I prefer **Fish**. Out of the box, Fish gives you auto-suggestions and syntax highlighting, and I’ve found its configuration much easier to understand, especially if you’re just starting out. Fish has also become popular enough that most tools now provide completion scripts for it. If you want to extend Fish, **fisher** is a great plugin manager.

The only real downside to Fish is that it’s not POSIX compliant, so a few commands don’t work the same way. For example:

- `sudo !!` won’t repeat the last command with `sudo`.
- You can’t start `ssh-agent` with `eval $(ssh-agent)`.

But for most day-to-day tasks, Fish is a pleasure to use, and the trade-offs are pretty minor.

### Zoxide

One of the major pain points of using terminals is navigating directories. For example, if I have a directory like `~/.config/kitty/`, the usual way to get there is:

```bash
cd ~/.config/kitty/
```

This works, but when you’re working on a large project or jumping between lots of folders, it quickly gets tedious. That’s where **zoxide** comes in. It gives you a smarter way to move around: zoxide remembers the directories you visit, so next time, you can just type a part of the name and jump straight there. For instance:

```bash
z kitty
```

That’s it. No need to type the full path every time. It might seem like a small thing, but it really does make a difference when you’re moving around your filesystem all day. Instead of thinking about the absolute path, you just give it a hint, and you’re there.

If you want to take this idea even further, there’s another tool called **broot**. It builds on the same concept, but changes the way you think about navigating files in the terminal. I wouldn’t call it as beginner-friendly as zoxide, but it’s worth a look if you want to experiment.

### Tmux

**Tmux** is what’s called a terminal multiplexer, which basically means you can run several programs in a single terminal window—either by splitting the window into panes or by using tabs. While most modern terminal emulators already let you split windows or open tabs, the real strength of tmux is its ability to keep your sessions alive, no matter what happens to your connection.

I can’t count how many times I’ve been working over SSH, only to have the connection drop in the middle of something important. Or maybe I accidentally close the terminal window and lose all my work. With tmux, you don’t have to worry about that. You can detach from a session at any time, close your terminal, and then come back later and pick up exactly where you left off. It’s saved me more than once from losing a half-finished script or a long-running process.

Recently, I’ve started using **Zellij** because it offers a better experience out of the box, but tmux is so widely used that you’ll find it preinstalled on most remote machines. That’s why it’s worth getting comfortable with tools like tmux, nano, and vim—sometimes, you just need to work with what’s already there.

If you spend any time working on remote servers, or you want a bit more control over your terminal sessions, tmux is one of those tools that’s good to have in your toolkit.

### Mise

When I first thought about writing this blog, the obvious choice for managing runtime versions that came to mind was **asdf**. It’s been around for a while and does the job well enough. However, after using **mise** for the past couple of weeks, I’ve found it to be a real gamechanger in my workflow.

One thing that always bugged me with asdf is the number of first-party plugins you get out of the box. If you want to manage runtimes for other languages, you end up installing community plugins. It’s not a huge deal, but the less configuration I have to worry about, the more I can focus on what I actually want to do. At the end of the day, I just need a tool that can manage different versions of Node or Java across different projects, and mise does that right out of the box—and then some.

Mise not only supports everything asdf does (thanks to its asdf backend), but also has support for other backends like `cargo`, `dotnet`, and `npm`. There’s no extra setup required for these; they just work.

But that’s not where it ends. Mise also lets you manage environment variables, secrets, and hooks directly.

One feature I find myself using a lot is **tasks**. You can define custom tasks in your `mise.toml` file and run them from anywhere in your project. For example, to set up a build task:

```toml
[tasks.build]
description = "Build project"
run = "npm run build"
```

And then you just run:

```bash
mise run build
```

Tasks are a powerful feature, and honestly, they deserve their own blog post. If you’re curious, you can read more about them [here](https://mise.jdx.dev/tasks/).

What I really appreciate is that you don’t have to be in the root of your project to run a task. If I’m buried somewhere in `foo/bar/baz`, I can still run `mise run build` and it’ll execute from the project root. It’s a small thing, but it takes away the mental overhead of remembering where you need to be to run a command.

You can also commit your `mise.toml` to your repository, so everyone on your team has the same setup without extra hassle.

### Chezmoi

Even if you try to keep your setup simple, there’s always some configuration you can’t avoid. Managing all your dotfiles and keeping them in sync across different machines can quickly turn into a headache. That’s where **chezmoi** comes in—it helps you centralize and version control your configuration files, so you don’t have to worry about missing settings or manual copy-pasting.

Getting started is straightforward:

```bash
chezmoi init                             # Initializes a local git repo to track your config files
chezmoi add ~/.config/fish/config.fish   # Add files you want to track
chezmoi edit ~/.config/fish/config.fish  # Edit files using your $EDITOR
chezmoi apply -v                         # Apply your changes
```

When you’re setting up a new machine, you just need to run:

```bash
chezmoi init https://github.com/$GITHUB_USERNAME/dotfiles.git
chezmoi apply -v
```

That’s it—your configuration is now set up just the way you like it.

Chezmoi also comes with a bunch of features that make managing your setup even easier:

- Password manager integration for keeping secrets safe
- File encryption for sensitive configs
- Declarative package installs so you can automate more of your environment

If you want to spend less time fiddling with configs and more time actually working, chezmoi is worth a look.

## Conclusion

There’s no shortage of CLI tools out there, and finding the right ones can make your daily workflow a lot smoother. The tools I’ve mentioned here have saved me time and frustration, and I hope they’ll do the same for you. Of course, everyone’s setup is a little different, so don’t hesitate to experiment and see what fits best with your way of working.

If you have a favorite CLI tool or a tip that’s made your life easier, feel free to share it—I’m always looking to learn something new.

Happy tinkering!
