# ed-docker-best-practice
by article https://medium.com/@fsegredo2000/docker-best-practices-6fa3de5f17cb

TL: DR

    Tips for Run Command, apt-get
    Image flavors
    COPY vs ADD
    Bonus (Dive)

“How can I improve my DockerFile?”

Today, we’re going to look at a few things that can help you improve your Dockerfile.

Reduce image size

This is a crucial aspect, as we do not wish to end up with excessively large images. By paying attention to how we install the packages that we need in the container, we can significantly reduce the size of the images.

    When using apt-get install certain package managers by default will install recommended packages, dependencies and debug packages (associated with the packages you pretend to install), which might not re required/usefull for your image, in that case we can cut down some size just by adding the flag — no-install-recommends

We can check the dependencies and recommended packages with the command:

apt-cache depends (PACKAGE)
