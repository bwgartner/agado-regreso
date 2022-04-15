# agado-regreso

Test Case/Suite and Reporting automation for performance regression test suites

![FlowChart](/images/AgadoRegreso-FlowChart.png)


## Assumptions / Current Limitations

- Only tested on SUSE Linux Enterprise ( SLE ) so far
  - for SLE requires subscription ( trial or purchased ) to access patches/updates
  - but mostly works on openSUSE as well

- Only tested with
  - **MMTests** suite
  - **Phoronix** suite
  - others may be added over time

## Process on target System Under Test ( SUT )

1. Install Linux operating system, ensure no updates repos are enabled or applied
   - suggest doing a very minimal ( pattern ) installation ( from ISO/JeOS )
   - ensure update repos are disabled during installation and also post-install
     - `zypper modifyrepo -d <repoName>`
   - install additional packages
     - `zypper install which wget unzip supportutils sudo rsync nfs-client git-core gcc gcc-c++`

2. Clone this GitHub repository
   - `git clone https://github.com/bwgartner/agado-regreso`
     - suggest dropping into "/tmp"
     - validate all settings ( in "agado-regreso.conf" ), reviewing the comments for suggestions
     - enable _transport_ and setup _transportDir_, suggest creating an NFS mount, in "/etc/fstab" via "serverHost:/path transportDir nfs user,noauto 0 0"
     - decide if you want to restrict the kernel versions to a gated start/last version ( for SLE, see https://scc.suse.com/packages for the available series )

3. Prepare test case suites
   - Clone the **MMTests** repository ( `git clone https://github.com/gormanm/mmtests` )
     - suggest dropping into "/tmp" ( otherwise you can edit agado-regreso.conf )
     - Validate ( in "agado-regreso.conf" ) that _mmtests=Y_ is enabled and _mmtestsDir_ settings match locations
     - select the desired test cases
       - see example set in "/tmp/agado-regreso/cache/mmtests-testCase.list" or pick others from the mmtests/configs directory
       - place your desired list/version in "${mmtestsLog}/testCase.list"
         - or just setup a single one until the update repos are re-enabled
     - validate with a sample run
       - `cd /tmp/mmtests ; ./run-mmtests.sh --no-monitor --config ./config stream`

   - Clone the **Phoronix** repository ( `git clone https://github.com/phoronix-test-suite/phoronix-test-suite.git` )
     - suggest dropping into "/tmp" ( otherwise you can edit agado-regreso.conf )
     - During SLE install or post-install : enable Web and Scripting Module
     - Validate ( in agado-regreso.conf ) that _phoronix=Y_ is enabled and _phoronixDir_ setting matches location
     - `cd /tmp/phoronix-test-suite`
       - `./install-sh`
       - install additional packages
         - `zypper install xdg-utils uuidd php7 php7-dom php7-json php7-zip php7-openssl php-gd make libxml2-devel libuuid-devel gcc gcc-c++ autoconf`
       - `phoronix-test-suite system-info`
       - `phoronix-test-suite list-tests`
         - select the desired test cases
           - see example set in "/tmp/agado-regreso/cache/phoronix-testCase.list"
           - place your desired list/version in "${phoronixLog}/testCase.list"
             - or just setup a single one until the update repos are re-enabled
       - `phoronix-test-suite batch-setup`
         - Save test results (Y)
         - Open the web browser (N)
         - Auto upload the results (N)
         - Prompt for test identifier (Y)
         - Prompt for test description (N)
         - Prompt for saved results (Y)
         - Run all test options (Y)
     - validate with a sample run
       - `phoronix-test-suite benchmark smallpt`

4. Do a manual wrapper run
   - `cd /tmp/agado-regreso ; sh -x ./bin/agado-regreso`
      - which will run the first test case scenario for this installation
      - collect results
      - if _transport_ is enabled, will mount, clone via rsync, then umount
      - by default, system will reboot ( so that each test starts from a similar state )

5. Setup automation
   - upon reboot run next test case
     - `cp /tmp/agado-regreso/systemd/agado-regreso* /etc/systemd/system`
     - `systemctl enable agado-regreso.timer`
   - re-enable update repos and prep for next iteration
     - to upgrade kernel, to apply recommended maintenance updates _muRec="Y"_, to apply security maintenance updates _muSec="Y"_
     - `zypper modifyrepo -e <repoName>`
   - prep for next iteration
     - `rm /tmp/agado-regreso/cache/SCOPE`
     - `rm /tmp/agado-regreso/cache/TESTCASE`
     - `rm /tmp/agado-regreso/cache/KERNELTIMESTAMP`
   - invoke `/usr/bin/sh -c 'cd /tmp/agado-regreso && ./bin/ripeti'`

6. When these cycles are completed, you should find test cases run across
   - initial, base kernel installation
     - maintenance update for security patches (if enabled)
     - maintenance update for recommended patches (if enabled)
     - kernel upgrade (if enabled)
   - look in "/tmp/agado-regreso/log" to see status at any point

## Process on reporting node

1. Install a system/VM/node with operating system
   - Preparation
     - get agado-regreso repo ... `git clone https://github.com/bwgartner/agado-regreso` agado-regreso repo
     - adjust "agado-regreso.conf" file
       - to match landing spot ( _functionDir_ )
       - setup NFS mount ( static / dynamic ) of the cached - _transportDir_ (to access the SUT results)

2. Address test suites

   - **MMTests**
     - Clone the mmtests repository ( `git clone https://github.com/gormanm/mmtests` )
       - install the following packages
         - `zypper install wget R-jsonlite gnuplot perl-List-BinarySearch-XS perl-Cpanel-JSON-XS gnuplot`
         - `cpan install List::BinarySearch File::Slurp`
     - adjust "agado-regreso.conf" file
       - to match where mmtests landed ( _mmtestsDir_ )

   - **Phoronix**
     - During SLE install or post-install : enable Web and Scripting Module
     - Clone the mmtests repository ( `git clone https://github.com/gormanm/mmtests` )
       - install the following packages
         - `zypper install php7 php7-zip php7-openssl php7-json php7-dom php-gd dejavu-fonts`
     - adjust "agado-regreso.conf" file
       - to match where Phoronix landed ( _phoronixDir_ )
2. Run reports
   - `cd /tmp/agado-regreso ; ./bin/raporto`

Feel free to file issues, submit pull requests or contact me with your feedback/suggestions
