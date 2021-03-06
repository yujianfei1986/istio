# Fuzz test for authorization policy

The directory includes the fuzz test for authorization policy to test potential policy bypass issues caused by path
normalization issues in Istio and Envoy.

This test is currently optional and not executed in the normal integration test. It is recommended to run the test
before each new Istio release and make sure all fuzz tests pass.

## Overview

![](overview.jpg)

The fuzz test uses specialized web vulnerability fuzzers (e.g. [dotdotpwn](https://github.com/wireghoul/dotdotpwn)) to generate a large number of  fuzzed requests. The
mutation is based on a predefined path (`/private/secret.html`) that should be rejected by the authorization policy.

The test backend uses real Web servers configured to serve at the above predefined path. If a fuzzer generated request
successfully gets the data at the predefined path, it means a policy bypass has happened otherwise it should have been
rejected with 403 by the authorization policy.

We use the real backends (e.g. Apache, Nginx) instead of the existing test apps in order to test against unknown new
normalization behaviors in real backend servers.

## Usage

1. Run the test with existing Istio deployment:

    ```bash
    go test ./tests/integration/security/fuzz/... -p 1 -v -tags="integfuzz integ" -test.run "TestFuzzAuthorization" \
      --istio.test.nocleanup --istio.test.env kube  --istio.test.kube.deploy=false - -timeout 30m \
      --istio.test.pullpolicy=IfNotPresent --istio.test.kube.loadbalancer=false --log_output_level=tf:debug
    ```

1. Wait for the test to complete and check the results, the test usually takes about 3 minutes.

## Next Steps

1. Support more real backends in addition to Apache and Nginx.

1. Support more fuzzers in addition to the dotdotpwn.

1. Support authorization ALLOW policy in addition to the DENY policy.
