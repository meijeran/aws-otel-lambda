# ADOT Collector Lambda Layer — with Decouple Processor

> **This is a community fork of [aws-observability/aws-otel-lambda](https://github.com/aws-observability/aws-otel-lambda).**
> It publishes a custom **collector-only** Lambda layer that includes the [decouple processor](https://github.com/open-telemetry/opentelemetry-lambda/tree/main/collector/processor/decoupleprocessor), which was not accepted into the upstream project.
> All other layers (Python, Java, Node.js, .NET) are unmodified — use the [official AWS layers](https://aws-otel.github.io/docs/getting-started/lambda) for those.

---

## Why this fork?

The **decouple processor** allows a Lambda function to finish its invocation *before* the OTel collector has finished exporting telemetry. This significantly reduces the latency impact of tracing on your Lambda response times, at the cost of slightly higher billed duration.

From the [decouple processor docs](https://github.com/open-telemetry/opentelemetry-lambda/tree/main/collector/processor/decoupleprocessor):

> _This processor decouples the receiver and exporter ends of the pipeline. This allows the lambda function to finish before traces/metrics/logs have been exported by the collector._

**When combined with the batch processor**, this is the recommended pattern for minimising both latency and export costs. Use `decouple` as the **last processor** in your pipeline:

```yaml
processors:
  batch:
  decouple:

service:
  pipelines:
    traces:
      receivers: [otlp]
      processors: [batch, decouple]
      exporters: [awsxray]
```

---

## Using the layer

Find the current layer ARNs for each region on the [**GitHub Releases page**](../../releases/latest).

Add the ARN for your region and architecture as a layer on your Lambda function:

```bash
aws lambda update-function-configuration \
  --function-name my-function \
  --layers arn:aws:lambda:us-east-1:YOUR_ACCOUNT:layer:aws-otel-collector-amd64-ver-0-138-0:1
```

Or in Terraform:

```hcl
resource "aws_lambda_function" "example" {
  # ...
  layers = ["arn:aws:lambda:us-east-1:ACCOUNT:layer:aws-otel-collector-amd64-ver-0-138-0:1"]
}
```

The layer is compatible with `provided.al2` and `provided.al2023` runtimes (Go, .NET, Rust, or any custom runtime).

---

## Supported components

| Receivers | Processors | Exporters | Extensions |
|-----------|-----------|-----------|------------|
| `otlpreceiver` | `decoupleprocessor` ✨ | `awsemfexporter` | `sigv4authextension` |
| | | `awsxrayexporter` | |
| | | `prometheusremotewriteexporter` | |
| | | `debugexporter` | |
| | | `otlpexporter` | |
| | | `otlphttpexporter` | |

✨ Added by this fork — not available in the official AWS layer.

---

## Releases

Releases are published automatically when a `v*` tag is pushed. Each release includes:
- Layer ARNs for all supported regions (both `amd64` and `arm64`)
- The layer zip files as downloadable artifacts (for self-hosting)

See the [Releases page](../../releases) for all versions.

---

## Releasing a new version (maintainer)

```bash
git tag v0.138.0
git push origin v0.138.0
```

This triggers the [Publish Collector Layer](.github/workflows/publish-collector-layer.yml) workflow, which builds, tests, and publishes the layer to all configured AWS regions.

---

## Keeping up with upstream

A [weekly sync workflow](.github/workflows/sync-upstream.yml) runs every Monday and either fast-forwards `main` from [aws-observability/aws-otel-lambda](https://github.com/aws-observability/aws-otel-lambda) or opens a PR if there are conflicts.

---

## Security

See [CONTRIBUTING](CONTRIBUTING.md#security-issue-notifications) for information on reporting security issues.

## License

Apache-2.0 — see [LICENSE](LICENSE).


As a downstream Repo of [opentelemetry-lambda](https://github.com/open-telemetry/opentelemetry-lambda), ___aws-otel-lambda___ publishes AWS managed OpenTelemetry Lambda layers that are preconfigured for use with AWS services and bundle the reduced ADOT Collector. Users can onboard to OpenTelemetry in their existing Lambda functions by adding these ready-made layers directly.
- Python layer [**aws-otel-python-<amd64|arm64>-ver-1-32-0**](https://aws-otel.github.io/docs/getting-started/lambda/lambda-python) contains OpenTelemetry Python `v1.32.0` and ADOT Collector for Lambda `v0.43.0`
- Nodejs layer [**aws-otel-nodejs-<amd64|arm64>-ver-1-30-0**](https://aws-otel.github.io/docs/getting-started/lambda/lambda-js) contains OpenTelemetry JavaScript Core `v1.30.0` with AWS Lambda Instrumentation `v0.50.3` and ADOT Collector for Lambda `v0.43.0`
- Java-Wrapper layer [**aws-otel-java-wrapper-<amd64|arm64>-ver-1-32-0**](https://aws-otel.github.io/docs/getting-started/lambda/lambda-java) contains OpenTelemetry Java `v1.32.0` and ADOT Collector for Lambda `v0.43.0`
- Java-Agent layer [**aws-otel-java-agent-<amd64|arm64>-ver-1-32-0**](https://aws-otel.github.io/docs/getting-started/lambda/lambda-java-auto-instr) contains AWS Distro for OpenTelemetry Java Instrumentation `v1.32.0` and ADOT Collector for Lambda `v0.43.0`
- Collector layer **aws-otel-collector-<amd64|arm64>-ver-0-117-0** contains ADOT Collector for Lambda `v0.43.0`. Compatible with [.NET](https://aws-otel.github.io/docs/getting-started/lambda/lambda-dotnet) and [Go](https://aws-otel.github.io/docs/getting-started/lambda/lambda-go) runtimes.



## Sample Apps
We provide [SAM and Terraform sample applications](sample-apps/) for AWS managed OpenTelemetry Lambda layers. You can play with these samples by the following:
1. Install AWS Cli, AWS SAM, Terraform, and configure AWS credentials correctly.
2. Checkout the current Repo by
   
   ```
   git clone --recurse-submodules https://github.com/aws-observability/aws-otel-lambda.git
   ```
   
3. Go to the language folder, such as `python`, `java`, run

   ```
   ./build.sh
   ```
4. Go to a sample application folder, such as `sample-apps/aws-sdk/deploy/wrapper/`.
    
5. Deploy sample application by,
       
    For Terraform sample application
    ```
    terraform init
    terraform apply -auto-approve
    ```
 To Deploy SAM sample application, navigate to `sample-apps/python-aws-sdk-aiohttp-sam/` and run.
    ```
    ./run.sh
    ```
## ADOT Lambda Layer available components

This table represents the components that the ADOT Lambda Layer will support and can be used in the [custom configuration for ADOT collector on Lambda](https://aws-otel.github.io/docs/getting-started/lambda#custom-configuration-for-the-adot-collector-on-lambda). The highlighted components below are developed by AWS in-house.

| Receiver       | Exporter                      | Extensions                  |
|----------------|-------------------------------|-----------------------------|
|otlpreceiver    | `awsemfexporter`              |`sigv4authextension`         |
|                | `awsxrayexporter`             |                             |
|                | prometheusremotewriteexporter |                             |
|                | debugexporter                 |                             |
|                | otlpexporter                  |                             |
|                | otlphttpexporter              |                             |

## Security

See [CONTRIBUTING](CONTRIBUTING.md#security-issue-notifications) for more information.

## Support 

Please note that as per policy, we're providing support via GitHub on a best effort basis. However, if you have AWS Enterprise Support you can create a ticket and we will provide direct support within the respective SLAs.

## License

This project is licensed under the Apache-2.0 License.
