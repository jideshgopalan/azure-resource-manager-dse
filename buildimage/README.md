# Build Image

This README describes how we build the VM that the templates use.  As a user of these templates, you should not need to do this.

## Create a VM
 
    azure group create DSE-Image-RG SouthCentralUS
    azure vm quick-create --vm-size Standard_DS14_v2 DSE-Image-RG dse-image SouthCentralUS Linux Canonical:UbuntuServer:14.04.4-LTS:latest image-160622
    ssh image-160622@104.214.108.221

## Install Java on the VM

    sudo su
    apt-get -y install software-properties-common
    add-apt-repository -y ppa:webupd8team/java
    apt-get -y update
    echo debconf shared/accepted-oracle-license-v1-1 select true | sudo debconf-set-selections
    echo debconf shared/accepted-oracle-license-v1-1 seen true | sudo debconf-set-selections
    echo "deb http://datastax%40microsoft.com:3A7vadPHbNT@debian.datastax.com/enterprise stable main" | sudo tee -a /etc/apt/sources.list.d/datastax.sources.list
    curl -L http://debian.datastax.com/debian/repo_key | sudo apt-key add -

## Download (but don't install) DataStax Enterprise

    dse_version=4.8.5-1
    apt-get -y update
    apt-get -y -d install dse-full=$dse_version dse=$dse_version dse-hive=$dse_version dse-pig=$dse_version dse-demos=$dse_version dse-libsolr=$dse_version dse-libtomcat=$dse_version dse-libsqoop=$dse_version dse-liblog4j=$dse_version dse-libmahout=$dse_version dse-libhadoop-native=$dse_version dse-libcassandra=$dse_version dse-libhive=$dse_version dse-libpig=$dse_version dse-libhadoop=$dse_version dse-libspark=$dse_version
 
## From the local Azure CLI 
    azure vm stop DSE-Image-RG dse-image
    azure vm generalize DSE-Image-RG dse-image
 
## Get the SAS URL

    azure storage account connectionstring show ben13709
    azure storage container sas create img rl 06/30/2016 -c "DefaultEndpointsProtocol=https;AccountName=ben13709;AccountKey=dhGblqecj7vrSHb7e1817WUDLa9LA7K3wltemU/riA=="

This creates a URL for the img:

    https://ben13709.blob.core.windows.net/img?se=2016-06-30T07%3A00%3A00Z&sp=rl&sv=2015-02-21&sr=c&sig=7XZ%2FZwWfW0utvr3fgnFvqytj9JxliN9DrzQ6iUh7wZs%3D

From this you'll have to infer the URL for the VHD.  In this case it is:

    https://ben13709.blob.core.windows.net/datastax2016522133331?se=2016-06-30T07%3A00%3A00Z&sp=rl&sv=2015-02-21&sr=c&sig=7XZ%2FZwWfW0utvr3fgnFvqytj9JxliN9DrzQ6iUh7wZs%3D
