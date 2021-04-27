**Note: This README is extracted from the original source project that I created. Please see [WARNING & COPYRIGHT NOTICE](../../../README.md#warning--copyright-notice) for more info.**

# GitLab

Improvements, Maintenance, Procedures - All Things GitLab

## GitLab Runner

Continuous deployment (CD) for all Solarix GitLab repositories is handled by an auto-scaling GitLab **Runner Manager** (`srn:ec2:solarix:core:gitlab::instance/runner-manager`). The Runner Manager is a [shared runner](https://docs.gitlab.com/ee/ci/runners/#shared-specific-and-group-runners). When a new deployment pipeline is started within the Solarix GitLab instance the Runner Manager makes a request for an available [`t3a.xlarge` EC2 Spot Instance](https://aws.amazon.com/ec2/spot/pricing/). This temporary spot instance is reserved for 60 minutes and is used to perform the current pipeline stages within its a Docker container. If the instance finished its work and a new pipeline deployment comes in, the Runner Manager will assign that work to the existing instance, rather than spin up a new instance.

On the other hand, if multiple pipelines must be executed simultaneously then multiple temporary instances are propagated. Once their 60-minute block expires, the instance is terminated.

All GitLab Runner Manager settings can be found in the [.gitlab/gitlab-runner/config.toml](.gitlab/gitlab-runner/config.toml) file, such as EC2 settings, timeout, price limits, etc

### Initial Docker Machine Instantiation

_Note: This section **only** applies for the creation of an entirely new Runner Manager, such as for use within a new GitLab instance._

GitLab Runner's docker machine executor has trouble during first-time creation as multiple processes fight with one another (see [docs](https://docs.gitlab.com/runner/executors/docker_machine.html#configuring-gitlab-runner)).

For this reason, it is recommended to manually create an appropriate docker machine instance with the same options used in `config.toml`:

```bash
$ docker-machine create --driver amazonec2 \
--amazonec2-ami ami-00363424a8fcdec0 \
--amazonec2-access-key <access-key> \
--amazonec2-secret-key <secret-key> \
--amazonec2-region us-west-2 \
--amazonec2-vpc-id vpc-04e2b51fab88610c0 \
--amazonec2-subnet-id subnet-063abf4059868fe6 \
--amazonec2-zone a \
--amazonec2-tags runner-manager-name,solarix-gitlab-runner-manager,gitlab,true,gitlab-runner-autoscale,true \
--amazonec2-security-group default \
--amazonec2-instance-type t3a.xlarge \
--amazonec2-block-duration-minutes 60 \
--amazonec2-request-spot-instance \
--amazonec2-spot-price 0.09 \
gitlab-docker-machine-1234567890
```

After this is successfully run for the first time the instance (spot request) can be safely removed. Future automated creations should work as normal.
