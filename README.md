# RHOS Perfscale CI
RedHat Openstack performance and scale CI consists of a set of jenkins pipeline jobs running on RHOS QE jenkins server. The jobs are triggered automatically on every new OSP puddle release passing passed_phase2.

## Job details
Every new job is created based on the combination of the OSP release version and the Neutron backend SDN type. To create a new job, you have to submit a patch on the `rhos-qe-jenkins` repository with your modification. Here is a [Mojo link](https://mojo.redhat.com/docs/DOC-1146679) for submit a patch on rhos-qe-jenkins. All the perfci jobs defined in the `jobs/DFG/perfscale/perfci/jobs/jobs.yaml` file. Currently the perfci contains 4 jobs

* [OSP13 with ovs](https://rhos-qe-jenkins.rhev-ci-vms.eng.rdu2.redhat.com/job/DFG-perfscale-PerfCI-OSP13-ovs/)
* [OSP13 with ovn](https://rhos-qe-jenkins.rhev-ci-vms.eng.rdu2.redhat.com/job/DFG-perfscale-PerfCI-OSP13-ovn/)
* [OSP16 with ovn](https://rhos-qe-jenkins.rhev-ci-vms.eng.rdu2.redhat.com/job/DFG-perfscale-PerfCI-OSP16-ovn/)
* [OSP16 with ovs](https://rhos-qe-jenkins.rhev-ci-vms.eng.rdu2.redhat.com/job/DFG-perfscale-PerfCI-OSP16-ovs/)

## What does each job do?
Each job is triggered automatically whenever the new puddle is released and passing the passed_phase2 for the respective OSP version. On each run, the job deploys the corresponding OSP version using [Jetpack](https://github.com/redhat-performance/jetpack) on the given baremetal nodes.  Jetpack is a tool to deploy openstack on baremetal.

After deploying OpenStack, the job will look for the DFG specific configuration variables file for each enabled DFG. The configuration variable files are available in the [perfci-vars](https://github.com/redhat-performance/perfci-vars) repository.

Post this configuration, the browbeat tests are run individually for each DFG on different stages and the result will be pushed to [ELK](http://10.9.76.205:5601/app/kibana) host.

## Flow of the Job
The below diagram explains the high level flow of each perfci job.

![flow](https://github.com/redhat-performance/perfci-vars/blob/master/images/job_flow.png)

RHOS QE jenkins will periodically poll the OSP puddle registry for any new releases, if it finds any new puddle release it will trigger the respective perfci job on the jenkins slave which is dedicatedly allotted for perfci jobs.

Upon the job trigger, the jenkins will pull the SCM where the Jenkinsfile script presents. In our case, the Jenkinsfile scripts are available at rhos-qe-jenkins repository at the jobs/DFG/perfscale/perfci/scripts directory and the supporting scripts are present at the infra/scripts/DFG/perfscale/perfci/scripts directory.

The job will continue to run on various stages, on stage one it will checkout the jetpack, configuration variables for Jetpack and browbeat for each DFG checked out from perfci-vars repository. On stage two it will deploy the openstack using jetpack. Post deployment the job will run the browbeat tests for all the enabled DFGs on different stages and the results are pushed to the ELK stack.

## How to enable new DFG
To add a new DFG in a Performance QE CI environment, You just need to do two simple steps. First, you have to add a configuration variable file for the DFG, the file name should be in <DFG_name>-config-vars.yml format. You need to submit a PR on perfci-vars repository to add your configuration file.

Secondly, you have to enable your DFG in the jobs’ DFG_LIST parameter before triggering the job. If you want to add your DFG permanently for every run, you need to submit a patch to [`rhos-qe-jenkins`](https://code.engineering.redhat.com/gerrit/gitweb?p=rhos-qe-jenkins.git) with the modified DFG_LIST on the respective jobs.

## How to add a new workload
To add a new workload for your DFG jobs, just add your new workload params on your <DFG>-config-vars.yaml and submit the change to the perfci-vars repository.

**Note:** when adding a new scenario to your DFG configuration variable file, make sure that the scenario is [supported](https://github.com/cloud-bulldozer/browbeat/tree/master/rally) in browbeat. In case it is not available on the Browbeat you can submit a patch to Browbeat to add your scenario. Here are the [steps](https://browbeat.readthedocs.io/contributing.html) to submit a patch on Browbeat.
