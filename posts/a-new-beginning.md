Welcome to the new look! Art of Coding will be changing from my personal
repository to something more professional and streamlined. This of course
includes the new blog you are currently reading.

## No more WordPress

Let's face it; WordPress is big. Way too big for a simple blog. The alternatives
I researched each had their own disadvantages, so I decided to make a small CLI
tool which helps manage a headless blog using Git. This blog is using it right
now - this post is hosted within a
[GitHub repository](https://github.com/Art-of-Coding/blog)!

[BlogGit](https://github.com/Art-of-Coding/bloggit) can now be installed as an
early alpha. It currently includes the ability to create a new post, and to
generate a Markdown file containing an overview of all posts. I expect to
iterate quickly, hopefully releasing a stable version before the end of the
year.

To get started with BlogGit, all you have to do is run the following command in
a fresh directory:

```bash
npx bloggit init
```

The wizard will walk you through setting up a Git repository and a `meta.json`,
the latter contains the blog's metadata.

## Started work on Nexus Engine

I am excited about my latest project.
[Nexus Engine](https://github.com/NexusEngine/nexus) will become a
next-generation text based browser game engine. Wow, that's a mouth-full!

The engine will be completely extensible using **mods**. The core of the engine
will be kept as abstract as possible, allowing the widest possible latitude for
developers to bring alive their dream projects.

The mods system has been kind of a headache, but I hope to get to something
workable soon. Since the mods API will be one of the most important parts of the
engine, I want to make sure it works well.

If you're a developer, you are welcome to contribute to either project!

Thank you for reading and I hope to see you somewhere!
