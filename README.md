# prpl Router CI

The complete prplOS Cram test suite is checked into this repository under
`.gitlab/tests/cram/`. The pipeline runs the generic tests, all tests for the
selected `DUT_BOARD`, and the post-test suite from this local source tree.

Configure these GitLab CI variables before starting `cram prplos`:

- `DUT_HOST` — DUT address
- `SSH_USER` — DUT SSH user
- `DUT_BOARD` — matching prplOS board directory, for example `wnc-freedom`

## Jenkins flow

The `Jenkinsfile` runs the same local prpl Cram suite:

1. Checkout the repository.
2. Validate the generic, board, and post-test directories.
3. Load the `prpl-dut-ssh` Jenkins credential.
4. Run generic tests, all tests for `DUT_BOARD`, and post-tests.
5. Publish the Cram JUnit results and archive the XML files.

The Jenkins agent must provide `python3 -m cram`. The SSH credential must be
an `sshUserPrivateKey` credential with ID `prpl-dut-ssh`.
