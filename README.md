# IBMi Export-Import 
The OpCon IBM i Agent Data Export/Import process enables the client to define an automatic process that exports LSAM master records groups from a test machine, then transfers them to one or many production machines based on the configuration of the OpCon Machine Group, and finally imports them on each machine using the LSAM Import process.

# IBM LSAM PTF Distribution from Test to Production
Similar to the LSAM automation rules data distribution process, above, the IBM LSAM PTF Distribution process enables the client to define an automatic process that exports LSAM PTFs (software patches - NOT the same as the IBM i OS PTFs) that were downloaded to a Test (or Source) partition from the SMA FTP server to be automatically transfered to one or many Production machines based on the configuration of an OpCon Machine Group.  The procedure finishes by executing the existing single command SMAPTFINS on each machine using the LSAM Import process.

# Prerequisites
* OpCon IBM i Agent 21.1

# Instructions
Please refer to [this](UserGuide-Automate-Exp-Imp.md) for detailed instructions about the LSAM Data Export-Import procedure.

Please refer to [this](UserGuide-Automate-LSAM-PTF.md) for detailed instructions about the LSAM PTF Distribution procedure.

# Disclaimer
No Support and No Warranty are provided by SMA Technologies for this project and related material. The use of this project's files is on your own risk.

SMA Technologies assumes no liability for damage caused by the usage of any of the files offered here via this Github repository.

# License
Copyright 2020 SMA Technologies

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at [apache.org/licenses/LICENSE-2.0](http://www.apache.org/licenses/LICENSE-2.0)

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.

# Contributing
We love contributions, please read our [Contribution Guide](CONTRIBUTING.md) to get started!

# Code of Conduct
[![Contributor Covenant](https://img.shields.io/badge/Contributor%20Covenant-v2.0%20adopted-ff69b4.svg)](code-of-conduct.md)
SMA Technologies has adopted the [Contributor Covenant](CODE_OF_CONDUCT.md) as its Code of Conduct, and we expect project participants to adhere to it. Please read the [full text](CODE_OF_CONDUCT.md) so that you can understand what actions will and will not be tolerated.
