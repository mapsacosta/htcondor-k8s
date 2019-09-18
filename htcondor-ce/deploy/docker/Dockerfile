FROM sl:7
MAINTAINER "Maria A." <macosta[at]fnal.gov>
ENV container docker

#Installing others

RUN yum update -y
RUN yum -y install openssh-server man wget git zip krb5-devel krb5-workstation strace vim

RUN mkdir .globus
RUN chmod 0700 .globus

#Install OSG3.4

RUN yum -y install https://repo.opensciencegrid.org/osg/3.4/osg-3.4-el7-release-latest.rpm \
                   epel-release \
                   yum-plugin-priorities && \
    yum -y install  \
                   osg-wn-client \
                   redhat-lsb-core \
                   singularity && \

#Install HTCondor (development)
RUN yum install -y wget sed ;\
    /usr/bin/wget http://research.cs.wisc.edu/htcondor/yum/repo.d/htcondor-testing-rhel7.repo -O /etc/yum.repos.d/condor.repo ;\
    /usr/bin/wget http://research.cs.wisc.edu/htcondor/yum/RPM-GPG-KEY-HTCondor -O /etc/pki/rpm-gpg/RPM-GPG-KEY-HTCondor ;\
    echo "priority=90" >> /etc/yum.repos.d/condor.repo ;\
    echo "gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-HTCondor" >> /etc/yum.repos.d/condor.repo

RUN yum install -y osg-ce-condor
RUN yum install -y condor-all

#COPY config/htcondor/config.d/* /etc/condor/config.d/
#COPY config/htcondor/certs/* /etc/condor/certs/
RUN touch /etc/condor/config.d/90_condor_test

ARG poolPwd 

RUN mkdir /etc/condor/password.d/ \
    && condor_store_cred add -c -p $poolPwd

RUN condor_token_create -identity htcondoece -token K8Sce -authz DAEMON,QUERY_JOB_ADS_WITH_AUTH

RUN mkdir /home/condor/ \
    && chown -R condor:condor /home/condor \
    && chmod +rw /home/condor  \
    && chmod -R a+rw /root/.condor/ 

#RUN rm -rf /etc/condor/password.d/
#Bug in HTCondor 8.9.2, so password folder has to be kept
RUN rm -rf /etc/condor/password.d/*
#If the token is being generated in docker, we still need to put in basic config so the token cnan be generated against the cprrect pool/collectors/etc so we remove it after generating the token.. 
RUN rm -rf /etc/condor/config.d

RUN systemctl enable condor
RUN systemctl enable condor-ce

ENV CMS_LOCAL_SITE=T1_US_FNAL_Disk

RUN yum clean all && \
    rm -rf /var/cache/yum/*

EXPOSE 9618

RUN yum -y install supervisor
COPY config/supervisor/supervisord.conf /etc/supervisor/conf.d/supervisord.conf

RUN yum clean all && \
    rm -rf /var/cache/yum/*

CMD ["/usr/bin/supervisord","-c","/etc/supervisor/conf.d/supervisord.conf"]
