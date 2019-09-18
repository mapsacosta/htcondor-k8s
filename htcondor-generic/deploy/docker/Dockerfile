FROM sl:7
ENV container docker

MAINTAINER "Maria A." <macosta[at]fnal.gov>


RUN yum -y update

#Installing CVMFS and others
RUN yum -y install openssh-server cvmfs man wget git zip krb5-devel krb5-workstation strace vim \
    && yum -y install https://ecsft.cern.ch/dist/cvmfs/cvmfs-release/cvmfs-release-latest.noarch.rpm

#Install OSG3.4
RUN yum -y install https://repo.opensciencegrid.org/osg/3.4/osg-3.4-el7-release-latest.rpm \
                   epel-release \
                   yum-plugin-priorities \
                   redhat-lsb-core

#Install HTCondor (development)
RUN yum install -y wget sed ;\
    /usr/bin/wget http://research.cs.wisc.edu/htcondor/yum/repo.d/htcondor-testing-rhel7.repo -O /etc/yum.repos.d/condor.repo ;\
    /usr/bin/wget http://research.cs.wisc.edu/htcondor/yum/RPM-GPG-KEY-HTCondor -O /etc/pki/rpm-gpg/RPM-GPG-KEY-HTCondor ;\
    echo "priority=90" >> /etc/yum.repos.d/condor.repo ;\
    echo "gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-HTCondor" >> /etc/yum.repos.d/condor.repo

RUN yum install -y condor-all

# Several ways to do this, config can be copied at build time into the image (not ideal esp when there's sensitive content involved)
# For testing purposes
#COPY config/htcondor/config.d/* /etc/condor/config.d/
#RUN touch /etc/condor/config.d/90_condor_test

# Config can be injected through Kubernetes configMaps. This requires some intermediate knowledge on Kubernetes and supposes the user deploying has admin access to the Kubernetes cluster where this image is being deployed.
# Probably the ideal way to go when doing production-level deployments
# In this case, only action required at Docker build time is to create the optional extra config file:
# And just adding an empty file for (if any) local condor knobs needed
RUN touch /etc/condor/config.d/90_condor_test

#Creating htcondor "home" directory, in k8s this is translated to a scratch volume for jobs to run
RUN mkdir /home/condor/ \
    && chown -R condor:condor /home/condor \
    && chmod +rw /home/condor

# Authentication: Several ways to do this too, assuming we generated a token outside this image and are copying it WITH the image:
#RUN echo "{THE_TOKEN}" > /etc/condor/tokens.d/TOKEN \
# && chown -R condor:condor /etc/condor/tokens.d/

# Another way to do it is by injecting the token through Kubernetes, in that case:
# 1. Create tokens.d folder with required permissions (Must be owned by condor)
# 2. Create an empty password.d directory (otherwise HTCondor will complain)
# 3. Fill up tokens.d folder as it better suits your needs?
RUN mkdir /etc/condor/tokens.d && mkdir /etc/condor/password.d \
    && chown -R condor:condor /etc/condor/tokens.d \
    && chmod +rw /etc/condor/tokens.d

#Making HTCondor shared port visible
EXPOSE 9620

RUN yum -y install supervisor
COPY config/supervisor/supervisord.conf /etc/supervisor/conf.d/supervisord.conf

CMD ["/usr/bin/supervisord","-c","/etc/supervisor/conf.d/supervisord.conf"]
