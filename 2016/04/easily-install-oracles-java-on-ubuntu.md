# Easily Install Oracle's Java On Ubuntu Linux

Friday, April 8, 2016

---

Hello world, today I have a quick "life hack" type thing for Ubuntu people.

Where I work, I am kind of the Linux guy, installing our software for clients who wish to run it on Ubuntu machines. \
The problem is that our software runs on Java, and some of the people either don't know how to install Java or have it installed in a weird way.

I was asked what was the easiest way to install Java, and here you go:
```bash
sudo add-apt-repository ppa:webupd8team/java
sudo apt-get update
sudo apt-get install oracle-java8-installer
# (agree to the two prompts)
# (wait for download to finish)
java -version
```

The first 2 commands installs the webupd8team Java repository and updates apt-get so you can install from there. \
The next command actually installs it, with a license agreement and a download. \
The last command just confirms that you have Java installed. \

<sub>Posted by [koppanyh](https://github.com/koppanyh) on 2016/04/08 at 10:33 AM PDT</sub>

<sub>[Permalink](https://blog.kh-labs.org/2016/04/easily-install-oracles-java-on-ubuntu)</sub>
