# Act 4 - Add an app

[Last time](../echo-chamber/echo-chamber.md) we learned how to modify our running cluster by
eliminating our beloved echo service.  We observed that we can easily modify the configuration
of our cluster, commit to our repository, and the changes are applied immediately.  We saw
that our configuration changes affect DNS entries, certificates, routing, and running containers.

This time instead of destroying resources, let's create something new.

# Echo's remains

To figure out what we need to do let's poke around in echo's remains, we left its config in our
repo, it's just not referenced anymore.

---

![vscode](img/vs-echo.jpg)

- Let's copy the echo directory into a new a `hello` directory:

---

![vscode](img/vs-hello.jpg)

Let's search for echo inside our hello folder:

---

![vscode](img/vs-hello-echo.jpg)

So we can see that we'll have to change at least these two files...

* Let's change `kubernetes/apps/default/hello/ks.yaml`

This defines our kubernetes app, if we replace echo with hello then we've defined a new hello
app!

---

![vscode](img/vs-replace-echo.jpg)

* Let's change `kubernetes/apps/default/hello/app/helmrelease.yaml`

 Here we're going to define what our app is supposed to do.  To start let's deploy a simple
 hello world web app!  Hold on, isn't that basically what echo did?  Absolutely!  But this is
 going to be so much better.


