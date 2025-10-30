##  Container Tooling Image Build Automation

This helps build container tool Images to support container build devops. 
The artifact here is for ubi9-haproxy container that can then be used to run containerized haproxy . 

To build this image usin the exisitng playbook run the following command from the root directory of the clone repository.

```bash 
ansible-playbook -vvv --ask-vault-pass -e tooling_image_name=devops-haproxy-rhel9 -e tooling_image_build_context_dir_prefix=haproxy-build -e tooling_image_containerfile=haproxy-build/context/Containerfile -e tooling_image_build_arg_base_image_name=registry.redhat.io/ubi9/ubi-minimal:latest -e registry_auth_config=<path-to-registry-authfile> -e push_tooling_image=true -e registry_host_fqdn=<registry-fqdn> -e local_repository=<image-repository> -e tooling_image_subrepository=<image-sub-repository> -e dir_bundle_location=<bundle-archive-location>  -e download_clients=flase build-devops-container-images.yml
``` 

Note that before running any playbook always make sure you properly set the appropriate variables as well as any related vaulted items. 
