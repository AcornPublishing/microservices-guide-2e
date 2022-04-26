# Appendix A: Installation of the Environment {#anhang-installation}

* The source code of the examples is available on Github. For access,
version control *git* must be installed, see
<https://git-scm.com/book/en/v2/Getting-Started-Installing-Git>. If
the installation was successful, a call of `git` in the command prompt
will work.

* The examples are implemented in Java. Therefore, *Java* has to be
installed. Instructions can be found at
  <https://www.java.com/en/download/help/download_options.xml>. Since
  the examples have to be compiled, a JDK (Java Development Kit) has
  to be installed. The JRE (Java Runtime Environment) is not
  enough. When the installation is completed, it should be
  possible to start `java` and `javac` in the command prompt. 

* The examples run in Docker containers. This requires an installation
of *Docker Community Edition*, see
<https://www.docker.com/community-edition/>. Docker can be called with
`docker`. This should work without errors after the installation.

* The examples require a lot of memory in some cases.
Therefore *Docker should have about 4 GB* available. Otherwise, it may
happen that Docker containers are terminated due to lack of memory.
Under Windows and macOS you can find the settings for this in the
Docker application under Preferences/Advanced. If there is not enough
memory, Docker containers are terminated. This is shown by the
entry `killed` in the logs of the containers.

* After installing Docker you should be able to call `docker-compose`.
If *Docker Compose* cannot be invoked, a separate installation is
necessary, see <https://docs.docker.com/compose/install/>.

Details regarding Docker are presented in
[chapter 5](#chapter-docker).
