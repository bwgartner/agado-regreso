# agado-regreso

Test Case/Suite and Reporting automation for performance regression test suites

![FlowChart](/images/AgadoRegreso-FlowChart.png)


## Assumptions / Current Limitations

- Only tested on SUSE Linux Enterprise ( SLE ) so far
  - for SLE requires subscription ( trial or purchased ) to access patches/updates
  - but might work on openSUSE as well

- Only tested with mmtests suite ( so far )
  - others may be added over time

## Process on target System Under Test ( SUT )

1. Install Linux operating system, ensure no updates repos are enabled or applied
   - suggest doing a very minimal ( pattern ) installation ( from ISO/JeOS )
   - install additional packages - wget, git-core, gcc, gcc-c++, rsync, supportutils
   - ensure update repos are disabled during installation and also post-install (zypper modifyrepo -d repoName )

2. Clone this GitHub repository ( git clone https://github.com/bwgartner/agado-regreso )
   - suggest dropping into /tmp
   - validate all settings ( in agado-regreso.conf )
   - enable transport and setup transportDir, suggest creating an NFS mount, in /etc/fstab via serverHost:/path transportDir nfs user,noauto 0 0

3. Clone the mmtests repository ( git clone https://github.com/gormanm/mmtests )
   - suggest dropping into /tmp ( otherwise you can edit agado-regreso.conf )
   - select the desired test cases ( place /tmp/mmtests/work/log/testCase.list )     - see sample set in /tmp/agado-regreso/cache/mmtests-testCase.list or just setup a single one until the update repos are re-enabled
   - validate with a sample run
   - cd /tmp/mmtests ; ./run-mmtests.sh --no-monitor --config ./config stream

4. Do a manual wrapper run
   - cd /tmp/agado-regreso ; sh -x ./bin/agado-regreso
   - which will run the first test case
   - collect results
   - if transport is enabled, will mount, clone, umount
   - system will reboot ( so that each test starts from a similar state )

5. Setup automation to continually run other test cases, then apply maintenance updates ( in security/recommended categories ) and then upgrade the kernel
   - cp /tmp/agado-regreso/systemd/ files to /etc/systemd/system
   - systemctl enable --now agado-regreso.timer
   - invoke /usr/bin/sh -c 'cd /tmp/agado-regreso && ./bin/ripeti'

6. When these cycles are completed, you should test cases run across
   - base kernel installation
   - maintenance update for security patches ( since no patches are enabled, this is just a re-run )
   - maintenance update for recommended patches ( since no patches are enabled, this is just a re-run )
   - kernel upgrade ( since no patches are enabled, this may not happen 

7. Re-enable update repos and prep for next iteration
   - rm /tmp/agado-regreso/cache/SCOPE
   - rm /tmp/agado-regreso/cache/TESTCASE
   - rm /tmp/agado-regreso/cache/KERNELTIMESTAMP
   - reboot, and SUT should upgrade the kernel, reboot and start the test cycle over

## Process on reporting node

1. Install a system/VM/node with operating system
   - git clone mmtest repo
     - install the following packages ... wget, gnuplot, perl-List-BinarySearch, perl-List-BinarySearch-XS, perl-Cpanel-JSON-X, R-rjson
   - git clone agado-regreso repo, adjusting config file
     - to match landing spot ( functionDir )
     - to match where mmtest landed ( mmtestsDir, mmtestsLog )
   - setup NFS mount ( static / dynamic ) of the cached - transportDir (to access the SUT results)

2. Run reports
   - cd /tmp/agado-regreso ; ./bin/raporto

Feel free to file issues, submit pull requests or contact me with your feedback/suggestions
