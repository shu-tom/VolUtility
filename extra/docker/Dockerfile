FROM ubuntu:20.04
MAINTAINER kevthehermit (https://github.com/kevthehermit/VolUtility)

# Version Vars
ENV YARA_VERSION        4.2.0
ENV SSDEEP_VERSION      2.14.1

# MaxMind GeoLite2
#ENV LICENSE_KEY xxxxxxxxxxxxxxxx

# Switch to user root
USER root

# set timezone
ENV TZ=Asia/Tokyo
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

# Install OS Dependancies
RUN apt update && apt install -yq \
 autoconf \
 automake \
 autopoint \
 curl \
 gettext \
 git \
 libimage-exiftool-perl \
 libtool \
 nano \
 pkg-config \
 python3-dev \
 python3-pip \
 sudo \
 software-properties-common

# Install packages from source
# Make Tmp Dir
RUN mkdir ~/tmp_build

# Install python2
RUN apt-get update && apt-get install -yq \
 python2 \
 python-dev

RUN cd ~/tmp_build && \
 curl https://bootstrap.pypa.io/pip/2.7/get-pip.py --output get-pip.py && \
 python2 get-pip.py

# Install Mongo
RUN \
curl -fsSL https://www.mongodb.org/static/pgp/server-5.0.asc | sudo apt-key add - && \
echo "deb https://repo.mongodb.org/apt/ubuntu focal/mongodb-org/5.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-5.0.list && \
apt-get update && \
apt-get install -yq mongodb-org

# Install Yara
RUN cd ~/tmp_build && \
 curl -sSL https://github.com/VirusTotal/yara/archive/refs/tags/v${YARA_VERSION}.tar.gz | \
 tar -xzC . && \
 cd yara-${YARA_VERSION} && \
 bash build.sh && \
 make install && \
 cd .. && \
 rm -rf yara-${YARA_VERSION} && \
 ldconfig

# Install Yara-python
RUN cd ~/tmp_build && \
 curl -sSL https://github.com/VirusTotal/yara-python/archive/refs/tags/v${YARA_VERSION}.tar.gz | \
 tar -xzC . && \
 cd yara-python-${YARA_VERSION} && \
 python3 setup.py build && \
 python3 setup.py install && \
 cd .. && \
 rm -rf yara-python-${YARA_VERSION}

# Install SSDEEP
RUN cd ~/tmp_build &&\
 curl -sSL https://github.com/ssdeep-project/ssdeep/releases/download/release-${SSDEEP_VERSION}/ssdeep-${SSDEEP_VERSION}.tar.gz | \
 tar -xzC .  && \
 cd ssdeep-${SSDEEP_VERSION} && \
 ./configure && \
 make install && \
 pip install pydeep && \
 cd .. && \
 rm -rf ssdeep-${SSDEEP_VERSION}

# Get the maxmind database for ip lookup
RUN cd ~/tmp_build && \
 curl -o GeoLite2-Country.tar.gz -sSL "https://download.maxmind.com/app/geoip_download?edition_id=GeoLite2-Country&license_key=${LICENSE_KEY}&suffix=tar.gz" && \
 tar -xzvf GeoLite2-Country.tar.gz && \
 mkdir -p /usr/share/GeoIP/ && \
 mv GeoLite2-Country_*/GeoLite2-Country.mmdb /usr/share/GeoIP/

# Install and Build libpff
RUN cd ~/tmp_build && \
 git clone https://github.com/libyal/libpff.git &&\
 cd libpff/ && \
 ./synclibs.sh && \
 ./autogen.sh && \
 ./configure --enable-python && \
 make && \
 make install && \
 ldconfig

# Install Volatility
RUN cd ~/tmp_build &&\
 git clone https://github.com/volatilityfoundation/volatility.git && \
 cd volatility && \
 python setup.py install

# Create Volutility User
RUN groupadd -r volutility && \
 useradd -r -g volutility -d /home/volutility -s /sbin/nologin -c "Volutility User" volutility && \
 usermod -a -G sudo volutility  && \
 mkdir /home/volutility && \
 chown -R volutility:volutility /home/volutility

# Get VolUtility
RUN cd /opt && \
 git clone https://github.com/kevthehermit/VolUtility.git && \
 chown -R volutility:volutility /opt/VolUtility

# Install PIP Requirements.
RUN cd /opt/VolUtility && \
 pip2 install -r requirements.txt

 # Clean Up
RUN rm -rf ~/tmp_build
RUN apt-get remove -yq \
 automake \
 autopoint \
 gettext \
 autoconf \
 gettext \
 pkg-config \
 libtool
RUN sudo apt-get clean

# Setup and Run
USER volutility
WORKDIR /opt/VolUtility
ADD start.sh start.sh
RUN mkdir ~/dbpath
RUN chmod 755 ~/dbpath
CMD /bin/bash /opt/VolUtility/start.sh
