[Documentation](https://bats-core.readthedocs.io/en/stable/)
[GitHub](https://github.com/bats-core/bats-core)

Bats-core is a [[TAP]]-compliant testing framework for Bash. It's a simple way to write tests to verify that your UNIX programs behave as expected.

## Usage
A test file is just a bash script with special syntax for defining test cases. Each test case is simply a function with a description.

```shell
#!/usr/bin/env bats

@test "addition using bc" {
	result="$(echo 2+2 | bc)"
	[ "$result" -eq 4 ]
}
```

Test cases consist of standard shell commands. Bats makes use of Bash's `errexit` (`set -e`) option when running test cases. If every command in the test case exits with a 0 status code, the test passes. 

## Filesystem Structure
We are aiming for the following filesystem structure:
```
src/
	script.sh
test/
	bats/
	test_helper/
		bats-support/
		bats-asset/
	test.bats
	...
```

From the project root:
```shell
git submodule add https://github.com/bats-core/bats-core.git test/bats
git submodule add https://github.com/bats-core/bats-support.git test/test_helper/bats-support
git submodule add https://github.com/bats-core/bats-assert.git test/test_helper/bats-assert
```

This will add submodules into your repo at the appropriate locations.

Write your tests in the `./test` directory with `.bats` extensions. 

### setup()
The setup function is called before each test. 

```shell
setup() {
	# get the directory of this file
	DIR="$( cd "$( dirname "$BATS_TEST_FILENAME" )" >/dev/null 2>&1 && pwd )"
    # make executables in src/ visible to PATH
    PATH="$DIR/../src:$PATH"
}
```

Each file can only define one setup function for all tests in the file. However, the setup functions can differ between different files. 

### load
A library is loaded by sourcing the `load.bash` file in its main directory.

Assuming that libraries are installed in `test/test_helper`, adding the following line to a file in the `test` directory loads the `bats-support` library.

```shell
load 'test_helper/bats-support/load'
```

_**Note:** The `load` function sources a file (with `.bash` extension automatically appended) relative to the location of the current test file._

_**Note:** The `load` function does NOT append a `.bash` extension automatically when loading a file using an absolute path._

Because we've loaded the `bats-support` and `bats-assert` modules we can now use `run` and `assert_output`

### run
Will execute the command it gets passed as parameters. It stores the stdout and stderr of the command it ran and stores it in $output and stores the exit code in $status and returns 0. The run command never fails the test and won't generate any additional output in the log of a failed test on its own. 

Marking the test as failed and printing context information is handled by consumers of $status and $output, referred to as an assertion.

### assertions
An assertion is a function that performs a test and returns 1 on failure or 0 on success. The assertion will also output relevant information on the failure. You can see more about them [here](https://github.com/bats-core/bats-assert)
[assert](https://github.com/bats-core/bats-assert#assert)
[refute](https://github.com/bats-core/bats-assert#refute)
[assert_equal](https://github.com/bats-core/bats-assert#assert_equal)
[assert_not_equal](https://github.com/bats-core/bats-assert#assert_not_equal)
[assert_success](https://github.com/bats-core/bats-assert#assert_success)
[assert_failure](https://github.com/bats-core/bats-assert#assert_failure)
[assert_output](https://github.com/bats-core/bats-assert#assert_output)
[refute_output](https://github.com/bats-core/bats-assert#refute_output)
[assert_line](https://github.com/bats-core/bats-assert#assert_line)
[refute_line](https://github.com/bats-core/bats-assert#refute_line)
[assert_regex](https://github.com/bats-core/bats-assert#assert_regex)
[refute_regex](https://github.com/bats-core/bats-assert#refute_regex)

### teardown()
The teardown function runs after each individual test in a file, regardless of test success or failure. Simliar to [setup()](#setup()), each `.bats` file can have its own teardown function which is the same for all tests in the file. 

```shell
teardown() {
	rm -f /tmp/bats-tutorial-project-ran
}
```

### skip
You can use this function in `setup()` or `@test` to skip a test. This can be useful for tailoring tests to run based on the environment. Skipped tests won't fail a test suite and are counted separately. No test command after `skip` will be executed. 

```shell
teardown() {
    : # Look Ma! No cleanup!
}

@test "Show welcome message on first invocation" {
    if [[ -e /tmp/bats-tutorial-project-ran ]]; then
        skip 'The FIRST_RUN_FILE already exists'
    fi

    run project.sh
    assert_output --partial 'Welcome to our project!'

    run project.sh
    refute_output --partial 'Welcome to our project!'
}
```

