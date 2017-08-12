# cf-ansible-conf

This is a collection of playbooks to build/scale CloudForms on all the providers CF supports. Currently, VMware, RHV, OpenStack & Amazon are complete & fully tested for standing up the initial DB server.

### These playbooks MUST be ran as root due to the fact that the rails environment does not execute correctly via sudo

I've made major changes since I first started this a couple of weeks ago.  Everything has been divided up into roles, and the general workflow is you that you create the instance from the specified provider (currently supports Amazon EC2, Red Hat OpenStack Platform, and VMware), then once that intance is up, the playbook will configure it as whatever type of CloudForms server you specify. So, if you want a DB server running on vsphere, you'd call the jritenour.vmware-cf.miq role, and then jritenour.cf-miq-db role to initialize it as a database server. If you look at the vmware-db-create playbook, you'll see an example of how to do this, as well as add the target vcenter as a provider in CloudForms once it's up.

The following "default" playbooks are setup:

*ec2-db-create* - Creates the DB server in EC2.  There's a bit of extra steps between the instance creation and DB init, as it has configure the instance to allow root over SSH, due the the way the cloudforms appliance_console_cli command works. Haven't tested creating workers this way yet.

*osp-db-create* - Creates the DB server on OpenStack.  I have note tested this yet since breaking everything into roles, but it *should* work.  Will test later in the week when I get access to an OpenStack environment again.

**NOTE:** Due to [BZ 1439373](https://bugzilla.redhat.com/show_bug.cgi?id=1439373), you'll need to import the VMware CF appliance OVF, power it up once, remove persistent udev rules and ssh host keys, then run

**echo "network: {config: disabled}" >> /etc/cloud/cloud.cfg.d/10_miq_cloud.cfg**

This will prevent cloud-init from wiping out the static network configuration on reboot.

*rhv-db-create* - Creates the DB server on Red Hat Virtualization.  Note that only RHV/Ovirt 4.x is supported at this time - RHEV 3.x uses a different set of ansible modules. I don't plan to support 3.x as it will be EOL relatively soon.

*vmware-db-create* - Creates the DB server in Vmware.

*vmware-ui-create* - Creates a worker to act as a UI server - enabled the UI, websocket & web service roles.

*vmware-wk-create* - Creates a worker in Vwmare & joins it to the specified DB server.  The playbook has static values currently assigned for the CF server roles assigned.  I'll tweak this later to allow choices from the following list:

automate, database_operations, database_owner, embedded_ansible, ems_inventory, ems_metrics_collector, ems_metrics_coordinator, ems_metrics_processor, ems_operations, event, git_owner, notifier, reporting, rhn_mirror, scheduler, smartproxy, smartstate, storage_inventory, storage_metrics_collector, storage_metrics_coordinator, storage_metrics_processor, user_interface, vmdb_storage_bridge, websocket,web_service

*vmware-full* - Calls all 3 vmware playbooks in order to create a distributed CloudForms environment. Next iteration will create a dedicated DB, UI and provider worker region

*azure-db-create* - not setup as a role yet.  This won't actually work until the azure_rm_virtualmachine module is modified to allow for custom images.  There is a PR open and should be resolved soon, hopefully

See roles/jritenour.cf-variables/defaults/vars_template for a listing of all the variables supported.

In addition, the VMware & RHV DB creation playbooks have been verified to work with the Embedded Ansible in CloudForms 4.5 - yes, you can actually use CloudForms to build CloudForms.  The only caveat is that you will want to add the EPEL repo to get the most recent version (2.3) of Ansible, as many of the cloud/virt modules have improved since 2.2, and my playbooks have some dependencies on these improvements.  You will also need any module specific python packages (eg pyvomi, boto, etc) to function properly.
