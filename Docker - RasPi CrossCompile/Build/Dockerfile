#Get latest ubuntu versio
FROM ubuntu:latest

#Variables used to synchronize files from Raspberry pi
ARG IP_ADDRESS=192.168.0.19
ARG USERNAME=pi
ARG PASSWORD=raspberry
ARG PORT=22
#Qt version
ARG QT_VERSION=5.12.3

#Update and install required softwares
RUN apt-get update -y && \
    apt-get install git wget python rsync cmake g++ openssh-client sshpass -y

COPY Build.sh /root/raspi/Build.sh
RUN chmod +x /root/raspi/Build.sh

COPY SyncFromDocker2Pi.sh /root/raspi/SyncFromDocker2Pi.sh
RUN chmod +x /root/raspi/SyncFromDocker2Pi.sh

COPY SyncFromPi2Docker.sh /root/raspi/SyncFromPi2Docker.sh
RUN chmod +x /root/raspi/SyncFromPi2Docker.sh

WORKDIR /root/raspi

RUN git clone https://github.com/raspberrypi/tools

#Make directory for rpi files
RUN mkdir sysroot sysroot/usr sysroot/opt

#Generate ssh-key. -q: Silence ssh-keygen, -N '': Provides a new passphrase for the key, -f id_rsa: Specifies name of the file in which to store the created key.
#Copy ssh-key to rpi. Sshpass will pass given password for later requests. StrictHostKeyChecking=no make ssh-copy-id ignore hostkey checking
#Sshpass: -p $PASSWORD given password
#ssh-copy-id: -o: (option) StrictHostKeyChecking=no (will pass hostkey checking), -f (Don't check if the key is already configured as an authorized key on the server),
# -i (keyfile) id_rsa (our generated keyfile) $USERNAME@$IP_ADDRESS (username@ipaddress with our variables)
RUN ssh-keygen -q -N '' -f /root/.ssh/id_rsa && \
    sshpass -p $PASSWORD ssh-copy-id -o StrictHostKeyChecking=no -f -i /root/.ssh/id_rsa -p $PORT $USERNAME@$IP_ADDRESS

#Sync rpi files. 

# --rsh: This option allows you to choose an alternative remote shell program to use for communication between the local and remote copies of rsync.
# -a (archive, ), -v (verbose, print what files are being transferred and summary), -z (compress, compresses the file data)
RUN rsync --rsh="ssh -p $PORT -o StrictHostKeyChecking=no" -avz $USERNAME@$IP_ADDRESS:/lib sysroot
RUN rsync --rsh="ssh -p $PORT -o StrictHostKeyChecking=no" -avz $USERNAME@$IP_ADDRESS:/usr/include sysroot/usr
##With: "; exit 0" we can make docker ignore errors
RUN rsync --rsh="ssh -p $PORT -o StrictHostKeyChecking=no" -avz $USERNAME@$IP_ADDRESS:/usr/lib sysroot/usr ; exit 0
RUN rsync --rsh="ssh -p $PORT -o StrictHostKeyChecking=no" -avz $USERNAME@$IP_ADDRESS:/opt/vc sysroot/opt

RUN wget https://raw.githubusercontent.com/Kukkimonsuta/rpi-buildqt/master/scripts/utils/sysroot-relativelinks.py && \
    chmod +x sysroot-relativelinks.py && \
    ./sysroot-relativelinks.py sysroot

RUN git clone git://code.qt.io/qt/qtbase.git -b ${QT_VERSION}

WORKDIR /root/raspi/qtbase

RUN ./configure -release -opengl es2 -device linux-rasp-pi-g++ -device-option CROSS_COMPILE=/root/raspi/tools/arm-bcm2708/gcc-linaro-arm-linux-gnueabihf-raspbian-x64/bin/arm-linux-gnueabihf- -sysroot /root/raspi/sysroot -opensource -confirm-license -skip qtwayland -skip qtlocation -skip qtscript -make libs -prefix /usr/local/qt5pi -extprefix /root/raspi/qt5pi -hostprefix /root/raspi/qt5 -no-use-gold-linker -v -no-gbm
RUN make -j3 && make install

WORKDIR /root/raspi/

RUN mkdir modules

WORKDIR /root/raspi/modules

RUN git clone git://code.qt.io/qt/qtdeclarative.git -b ${QT_VERSION} && \
    git clone git://code.qt.io/qt/qtquickcontrols.git -b ${QT_VERSION} && \
    git clone git://code.qt.io/qt/qtquickcontrols2.git -b ${QT_VERSION}

WORKDIR /root/raspi/modules/qtdeclarative

RUN ~/raspi/qt5/bin/qmake && \
    make -j3 && \
    make install

WORKDIR /root/raspi/modules/qtquickcontrols

RUN ~/raspi/qt5/bin/qmake && \
    make -j3 && \
    make install

WORKDIR /root/raspi/modules/qtquickcontrols2

RUN ~/raspi/qt5/bin/qmake && \
    make -j3 && \
    make install

WORKDIR /root/raspi/

RUN rsync --rsh="ssh -p $PORT -o StrictHostKeyChecking=no" -avz /root/raspi/qt5pi $USERNAME@$IP_ADDRESS:/usr/local

#
CMD tail -f /dev/null

#ENTRYPOINT [ "bash" ]