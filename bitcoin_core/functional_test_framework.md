## Functional Test Framework

(Current as of commit 98005b6a17907c4f7bdcf802ee96c274492f902a).

See https://github.com/bitcoin/bitcoin/tree/master/test

Functional tests are present in `test/functional` and are executed by running `test_runner.py` (e.g. `test/functional/test_runner.py --cachedir=/mnt/tmp/cache --tmpdir=/mnt/tmp`).

`test_runner.py` runs test framework unit tests (in `TEST_FRAMEWORK_MODULES`), and most functional tests are listed in the `BASE_SCRIPTS` list.

After parsing command line arguments, setting up a temporary test directory, performing some sanity checks, the list of functional tests that will be executed is built.  If the user specified specific tests, the `test_list` is built with those test included.  If not, the standard base set of functional tests (or base + extended, if CLI arg specifies) is added to `test_list`.  Afterward, excluded test cases (CLI specified) are removed from `test_list`.

After some additional sanity checking (e.g. that chosen test scripts follow convention), `run_tests()` is called.  `run_tests()` performs some basic checks (e.g. for warnings about `bitcoind` running and a cache directory being used), then runs the unit tests for the Test Framework modules.

`job_queue` is then created, and in a loop `job_queue.get_next()` is called, which creates subprocess jobs for the tests, and returns test results from finished tests.

