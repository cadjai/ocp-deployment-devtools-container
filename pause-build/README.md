##  Container Tooling Image Build Automation

This helps build container tool Images to support container build devops. 
The artifact here is for ubi9-pause container that can then be used to run pods with multiple containers running inside it. 

To build this image usin the exisitng playbook run the following command from the root directory of the clone repository.

```bash 
ansible-playbook -vvv --ask-vault-pass -e tooling_image_name=container-tooling-ubi9-pause -e tooling_image_build_context_dir_prefix=pause-build -e tooling_image_containerfile=pause-build/context/Containerfile -e tooling_image_build_arg_base_image_name=registry.redhat.io/ubi9-micro:latest -e registry_auth_config=<path-to-registry-authfile> -e push_tooling_image=true -e registry_host_fqdn=<registry-fqdn> -e local_repository=<image-repository> -e tooling_image_subrepository=<image-sub-repostiroy> -e dir_bundle_location=<bundle-archive-location>  -e download_clients=flase build-devops-container-images.yml

``` 

Note that before running any playbook always make sure you properly set the appropriate variables as well as any related vaulted items. 
