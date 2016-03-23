The biggest impact on data science right now is not coming from a new algorithm or statistical method. It’s coming from Docker containers. Containers solve a bunch of tough problems simultaneously: they make it easy to use libraries with complicated setups; they make your output reproducible; they make it easier to share your work; and they can take the pain out of the Python data science stack.Bell_jar_apparatus

We use Docker containers at the heart of Kaggle Scripts. Playing around with Scripts can give you a sense of what you can do with data science containers. But you can also put them to work on your own computer, and in this post I’ll explain how.

Why use containers?
Containers are like ultralight virtual machines. When you restore a normal VM from a snapshot it can take a minute or so to get going, but Docker containers start up in roughly a millisecond. So you can run something inside a container just like you’d run a native binary. Every time you restart the container, its execution environment is identical, which gives you reproducibility. And containers run identically on OS X, Windows and Linux, so collaborating and sharing becomes much easier than before.

Personally, I think the best thing about containers is that they eliminate the pain of using Python for data science. R and Python are both great for statistics, each with its own strengths and weaknesses, but one striking difference between them is in how they handle libraries and packages. R’s install.packages() mechanism works very smoothly, and conflicts between packages are rare. If you come across a new piece of work that uses a library you don’t have on your system, you can install it from CRAN and be underway in a few moments.

What a contrast with Python. In the Python world, a typical workflow would be something like this: notice that you need libraryX, so call pip install X, which also installs dependencies A, B and C. But B already exists on your system via easy_install, so pip cancels itself but only partially removes the new stuff, then import B refuses to work ever again. Or you discover that Crelies on a later build of numpy, which you install, only to discover that libraries Y and Z are linked to an older numpy library that just got stomped on. And so on, and so on.

Python installations gradually accrete problems like this, with conflicts building up between libraries, and further conflicts between separate Python setups on the same system. The virtualenv system helps a little, but in my experience it just delays the crash. Eventually you reach a point where you have to completely reinstall Python from scratch. And that’s not to mention the hours you can spend getting a new library to work.

If you use Python in a container instead, all those problems vanish. You only have to invest time once in setting up the container: once the build is complete, you’re all set. In fact, if you use one of Kaggle’s containers, you don’t need to worry about building anything at all. And you can try out new packages without any hassles, because as soon as you exit a container session, it resets itself to a pristine state.

What’s in them exactly?
To run Kaggle Scripts, we put together three Docker containers: kaggle/rstats has an R installation with all of CRAN and a dozen extra packages, kaggle/julia has a recent build of Julia 0.5 with a set of data science libraries installed, andkaggle/python is an Anaconda Python setup with a large set of libraries. To see the details of what’s inside, you can browse the Dockerfiles that are used to build them, which are all open source. We had to split them up into several parts so we could auto-build them on Docker Hub: here are links to Python part 1, part 2, part 3; rcran0 to 22, and rstats; and Julia part 1, part 2.

One side note: we only support Python 3. I mean come on, it’s 2016.

How to get started
Here’s a recipe for setting up the Python container locally. These exact steps are for OS X, but the Windows or Linux equivalents are easy to figure out if you rtfm.

Step one is to head over to the Docker site and install Docker on your system. They’ve made the install process very easy, so that shouldn’t take more than the twinkling of an eye.

Step two: the default install creates a Linux VM to run your containers, but it’s quite small and struggles to handle a typical data science stack. So make a new one, which in this example I’ll call docker2.

1
$ docker-machine create -d virtualbox --virtualbox-disk-size "50000" --virtualbox-cpu-count "4" --virtualbox-memory "8092" docker2
Obviously, you can tailor the disk-size, cpu-count and memory numbers for your system. Step three: start it up.

1
2
$ docker-machine start docker2
$ eval $(docker-machine env docker2)
Later, if you open a new terminal window and Docker complains about Cannot connect to the Docker daemon. Is the docker daemon running on this host? then rerunning those two lines should sort it out.

Step four: pull the image you want to use.

1
$ docker pull kaggle/python
You’re now at a point where you can run stuff in the container. Here’s an extra step that will make it super easy: put these lines in your .bashrc file (or the Windows equivalent)

1
2
3
4。
5
6
7
8
9
10
kpython(){
  docker run -v $PWD:/tmp/working -w=/tmp/working --rm -it kaggle/python python "$@"
}
ikpython() {
  docker run -v $PWD:/tmp/working -w=/tmp/working --rm -it kaggle/python ipython
}
kjupyter() {
  (sleep 3 && open "http://$(docker-machine ip docker2):8888")&
  docker run -v $PWD:/tmp/working -w=/tmp/working -p 8888:8888 --rm -it kaggle/python jupyter notebook --no-browser --ip="\*" --notebook-dir=/tmp/working
}
Now you can use kpython as a replacement for calling python, ikpython instead of ipython, and run kjupyter to start a Jupyter notebook session. All of them will have immediate access to the complete data science stack that Kaggle assembled.

I hope you enjoy using these containers as much as I have. And let me just add one more plug for Kaggle Scripts—it’s a great way to share ideas and show off what you’ve made.

P.S. Here’s some more detail on how the .bashrc entries work. The three commands are Bash functions. The syntax docker run ... kaggle/python X will execute command X inside the Kaggle Python container. You give the container session access to the directory that you’re currently in by adding -v $PWD:/tmp/working, and for convenience -w=/tmp/working makes the session start in that working directory. The --rm switch tidies up the container session after you exit. By default, Docker sessions hang around in case you want to do a post-mortem on them. Finally, the -it means that the container’s stdin, stdout and stderr will be attached to your terminal. There are many other options that you can use, but I’ve found those to be the most useful.
