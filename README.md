## OCP Deployment Devops Container Image Build Automation

We are trying to adopt a container based approach to running our day zero deployment automation. To accomplish this we will build a container image that will replicate steps performed withiin a bastion or jumphost where cluster deployment steps as well as day two playbooks are run. 
This is the first step in a long journey to get us ready to start using Ansible Builder Execution Images, which can easily be swapped in once we get AAP subscription. Until then we are simulating the builder process to build our own custom executon environment. 

The playbook and files in this directory allow us to build a new version of the deployment devops image whenever an update is needed (e.g. new package is required, new binaries need to be included, new ansible collections are required, ...). 

The main directory containes several Containerfile that can be used to perform the image build. When the playbook is run without any argument the default file is used which uses a podman based image to perform the build. The playbook also contains several optional variables that enable the controlling of the build. For example if you have a different context directory to use you can set the path using tooling_image_build_context_dir variable, to change the base image tooling_image_build_arg_base_image_name. All of the optional variables are set with empty string the the clients.yml variable file. 

Before running this playbook you will need the clients either already staged or you will need to set the download_clients boolean to true to have the playbook download the clients for you. 
This playbook can only be run on an Internet connected bastion host with access to the location where the clients are hosted. If not please ensure you can download the exisiting clients from the binary repository (e.g. artifactory) and stage them appropriately (the client binaries are expected under context/_build/clients directory). 

Once the clients have been staged or you have updated the clients.yml variable file to ensure that the clients will be downloaded, you need to make sure that you also update any necessary artifacts. For example if more ansible collections need to be included update the requirements.yml file included under the context/_build directory to reflect your intent. Also it is assumed that the collections can downloaded within the container. If not they will need to be staged under the context/_build directory as well. That will require a modification to the Containerfile so that the collection are installed from a local directory within the build container aftr having been copied in there similar to how the client binaries are being staged. 
Once all of that is done, run the playbook as follows: 

```bash 
ansible-playbook --ask-vault-pass -v build-devops-container-images.yml 
``` 
To override the default Containerfile as well as associated container image name, use the following command where we are using the ubi8 Containerfile  
```bash
ansible-playbook --ask-vault-pass -v -e download_clients=false -e tooling_image_name=ubi8-bastion -e tooling_image_containerfile=context/Containerfile-ubi8 build-devops-container-images.yml 
```
To override the default Containerfile as well as associated container image name and pushing the resuling image to a registry , use the following command where we are using the ubi8 Containerfile  

```bash
ansible-playbook --ask-vault-pass -v -e download_clients=false -e tooling_image_name=devops-ee-supported-rhel9 -e tooling_image_containerfile=ee-build/context/Containerfile-ee-supported-rhel9 -e dir_bundle_location=<path-to-save-image-bundle> -e registry_host_fqdn=<registry-fqdn> -e registry_auth_config=<path-to-registry-authfile> -e push_tooling_image=true -e local_repository=<repository> -e tooling_image_subrepository=<image-sub-repository>  build-devops-container-images.yml 
```
Note that before running any playbook always make sure you properly set the appropriate variables as well as any related vaulted items. 


Note that if you are fully disconnected and don't have access to a private automation hub and a pip server you can still build the custom EE,  provided that you have all of the collections and pip packages available locally. 

if you are doing so you can update your containerfile to reflect the local install by copying the collections into the builder using the `COPY` instruction and similarly copying the pip packages into the builder using the same instruction. 
Once done you can then update the appropriate command to reflect the local install using for example `/usr/bin/python3 -m pip install <local-path-to-pip-packages>` to install the pip packages or `RUN pushed /output/collections/ && ANSIBLE_GALAXY_DISABLE_GPG_VERIFY=1 ansible-galaxy collection install $ANSIBLE_GALAXY_CLI_COLLECTION_OPTS -r requirements.yml --offline --collections-path "/usr/share/ansible/collections" && popd` to install the locally available collections. See the `ee-build/context/Containerfile-ee-supported-rhel9-local` containerfile as a sample of how to update the containerfile. 

