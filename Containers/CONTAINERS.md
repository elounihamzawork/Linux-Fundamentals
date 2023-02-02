# What the heck are containers?

> Containers are a way of executing processes with isolation.

With containers, we can run a process and its subprocesses in isolation from the underlying system and all other containers. Each container has its own view on the operating system, its own filesystem, and access to an individual subset of resources (such as memory and CPU).

# How to run processes in isolation
Running processes in isolation is possible via three Linux technologies: **changing the root filesystem (chroot)**, **namespaces (unshare)**, and finally **control groups (cgroups)**.

- By changing the root we can isolate the process filesystem and protect the system filesystem from unwanted changes.
- Namespaces create a sliced view on the system resources such as process IDs, mount points, networks, users, etc.
- Control groups can restrict various computer resources such as memory, CPU, or network traffic.

### Change root (chroot)
A chroot is a way of isolating applications from the rest of your computer, by putting them in a jail. This 
is particularly useful if you are testing an application which could potentially alter important system files, or which may be insecure.

**Basic Concepts**
A chroot is basically a special directory on your computer which prevents applications, if run from inside that directory, from accessing files outside the directory. 
In many ways, a chroot is like installing another operating system inside your existing operating system.
Technically-speaking, chroot temporarily changes the **root directory (which is normally /)** to the chroot directory (for example, /var/chroot). 
**As the root directory is the top of the filesystem hierarchy**, **applications are unable to access directories higher up than the root directory**, and so are isolated from the rest of the system. 
This prevents applications inside the chroot from interfering with files elsewhere on your computer.
Note that it is possible for software from outside the chroot to access files inside the chroot.

##### Uses of chroots
The following are some possible uses of chroots:

- Isolating insecure and unstable applications
- Running 32-bit applications on 64-bit systems
- Testing new packages before installing them on the production system
- Running older versions of applications on more modern versions of Ubuntu
- Building new packages, allowing careful control over the dependency packages which are installed

##### Creating a chroot
This section provides instructions on creating a basic chroot. For more advanced chroots, see [link](https://help.ubuntu.com/community/DebootstrapChroot)

1. Install the schroot and debootstrap packages.

2. As an administrator (i.e. using sudo), create a new directory for the chroot. In this procedure, the directory /var/chroot will be used. To do this, type sudo mkdir /var/chroot into a command line.

3. As an administrator, open /etc/schroot/schroot.conf in a text editor. Type cd /etc/schroot, followed by gksu gedit schroot.conf. This will allow you to edit the file.

4. Add the following lines into schroot.conf and then save and close the file. Replace your_username with your username.
    ```
    [lucid]
    description=Ubuntu Lucid
    location=/var/chroot
    priority=3
    users=your_username
    groups=sbuild
    root-groups=root
    ```

5. Open a terminal and type:
    ```
    sudo debootstrap --variant=buildd --arch i386 lucid /var/chroot/ http://mirror.url.com/ubuntu/

    ```
This will create a basic 'installation' of Ubuntu 10.04 (Lucid Lynx) in the chroot. It may take a while for the packages to be downloaded.
**Note:** You can replace lucid with the Ubuntu version of your choice.
**Note:** You must change the above mirror.url.com with the URL of a valid archive mirror local to you.

#### Setting-up the chroot
There are some basic steps you can take to set up the chroot, providing facilities such as DNS resolution and access to /proc.
**Note:** Type these commands in a shell which is outside the chroot.
1. Type the following to mount the /proc filesystem in the chroot (required for managing processes):
    ```
    sudo mount -o bind /proc /var/chroot/proc
    ```
2. Type the following to allow DNS resolution from within the chroot (required for Internet access):
   ```
   sudo cp /etc/resolv.conf /var/chroot/etc/resolv.conf
   ```

**Note:** The obvious problem is that we can actually see all the host processes, not only the processes which run within the container.

Chroot alone is not enough to run processes in isolation. We have to reach for another tool: namespaces.

### Namespaces

- Namespaces are an **isolation mechanism**. Their main purpose is to isolate containers running on the same host so that these containers cannot access each other’s resources.
- Namespaces can be composed and nested
