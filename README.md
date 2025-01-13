# Instructions to run Universal Robots Simulator on Mac Apple Silicon/ARM 

## Setup Considerations
As of Jan 2025, [Universal Robots E-Series Offline Simulator](https://www.universal-robots.com/download/software-ur20ur30/simulator-linux/offline-simulator-e-series-and-ur20ur30-ur-sim-for-linux-5200/) is a software written in Java with some x86 32 bits binary (URControl). If it was solely in Java, it could run on ARM processors just by having the right Java JVM on ARM, but the x86 32 bits binary requires emulation of the x86 processor for the ARM computers. [VirtualBox](https://www.virtualbox.org), [VMWare](https://www.vmware.com) and [Parallels](https://www.parallels.com) did not have any x86 emulation at that time. We tried many possibilities such as using the [UTM](https://getutm.app) VM software on Mac with QEMU x86 emulation, but it was way way too slow. We tried also UTM with Apple Virtualization and Rosetta emulation, but it almost worked (URSim started, but got stuck).  We had more success with Docker and it was pretty easy once we found the solution to use the Docker image provided by Universal Robotics.

Therefore these instructions are based on running the Docker image provided by Universal Robotics, tell docker it is a amd64 image, and let Apple Rosetta do the x86 emulation. The performance, while far from a native ARM application, was sufficient. Hence the instructions below.

## Steps
- Install Rosetta on your Mac, which does Intel x86 emulation on ARM:
	- In Terminal: `softwareupdate --install-rosetta --agree-to-license`
- Install Docker 
	- either [Docker Desktop](https://www.docker.com/products/docker-desktop/) for Mac - Apple Silicon. Install and launch it. (No need to have a user logged into the Docker network)
	- or in MacOS Terminal> `brew install docker`
- Create a directory on your mac (example used: "Documents/programs") to be shared with the Docker instance. Learn the importance of this step in the 
Usage Considerations section below.
- In the Docker Terminal window (button on the bottom right of the main window) or in the MacOS Terminal if you use brew, type (in one line):
	- `docker run --rm -it -p 5900:5900 -p 6080:6080 -v "${HOME}/Documents/programs:/ursim/programs" —platform=linux/amd64 universalrobots/ursim_e-series`
	- the x86 to ARM emulation is done by the underlying Rosetta but we tell Docker 
	that the image is an amd64 with the `--platform` argument. One can use the exact same instructions here for Mac Intel, Windows or Linux by just not adding the `--platform` argument.
	- the -p arguments are used to expose ports from the docker to the host computer(aka your mac). 5900 is for VNC. 6080 is for http. 
	- the -v argument is to map a directory within the docker with a directory on the mac.
- now that the URSim software is launched in docker, we access it using a remote screen sharing program VNC.
- Find the IP address of your Mac: Settings->Wifi (or Network)->Details->TCP/IP->IP Address. Example: 192.168.1.242
- Run a VNC client. If you don't have one, you can download and install [RealVNC Viewer for Desktop for MacOS](https://www.realvnc.com/en/connect/download/viewer/)
- in the VNC client, connect to `192.168.1.242:5900`. Replace 192.168.1.242 by your IP address.
- URSim should show on your VNC client screen.

## Usage Considerations
Docker are lightweight ephemeral instances. When a docker instance is terminated (by the user typing Ctrl-C or accidently), all files are gone and cannot be retrieved. When launching a new docker instance, it starts a fresh new one. Therefore, all user files, profiles, state of the software are lost when an instance is terminated. To mitigate that, one can add directory sharing between the docker instance and the host (your Mac) or use Docker volumes. The former is what is proposed in these instructions. Therefore, before terminating your docker instance, make sure all the files you need next time are saved in the shared directory (within docker, per the instructions above, this is the /ursim/programs directory).

Instead of using VNC to access the URSim software, a web browser, such as Safari or Chrome, can be used by going to the url: `http://192.168.1.242:6080`. Replace 192.168.1.242 by your IP address.

Note that if the host computer (your mac) moves to a different network, such as when a laptop is using a different Wifi network, then the IP address changes. Therefore, while your docker may be still running, you will need to find your new IP address and reconnect your VNC client or web browser to that new IP address.

## More Information
Additional information for more tailored docker setup is available at the [URSIM docker image information](https://hub.docker.com/r/universalrobots/ursim_e-series)

## Comments
Please send any comments on these instructions to mailto:marc.blanchet@viagenie.ca or create issues or pull request on this repo.

Thanks to Julie Blanchet and Guillaume Blanchet for hints and testing of these instructions.