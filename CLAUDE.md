# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This project creates a CloudFormation template for a serverless, no-code solution that:
1. Subscribes to an SNS topic for AWS WAF Managed Rules updates
2. Transforms notifications into a format compatible with AWS Chatbot
3. Delivers formatted notifications to Slack/Teams via AWS Chatbot

## Architecture Patterns

This is a serverless event-driven notification pipeline using:
- SNS Topics for receiving WAF Managed Rules update notifications
- EventBridge/Lambda (or Step Functions) for message transformation
- AWS Chatbot for delivering notifications to collaboration platforms

## CloudFormation Development

When creating CloudFormation templates:
- Use YAML format for better readability
- Follow AWS best practices for resource naming and tagging
- Include parameters for configurable values (SNS topic ARN, Chatbot channel ARN, etc.)
- Add outputs for important resource ARNs

## MCP Server Usage

This project is configured with AWS-specific MCP servers. When working with AWS services:

1. **For AWS service questions and architecture guidance**: Use `mcp__awslabs-core-mcp-server__prompt_understanding` first
2. **For CloudFormation operations**: Use `awslabs.cfn-mcp-server` tools (configured in read-only mode)
3. **For AWS documentation**: Use `aws-knowledge-mcp-server` or `awslab-aws-documentation-mcp-server`
4. **For CDK patterns**: Use `awslab-cdk-mcp-server` for guidance on AWS constructs

## Key AWS Services

- **AWS WAF**: Source of managed rules update notifications
- **Amazon SNS**: Publishing and subscribing to notifications
- **AWS Lambda** (if needed): For complex message transformations
- **Amazon EventBridge**: For event routing and simple transformations
- **AWS Chatbot**: For Slack/Teams integration

## Testing Approach

- Use AWS CloudFormation validate command for template syntax
- Deploy to a test environment first
- Test with sample SNS messages matching WAF update format
- Verify AWS Chatbot delivers messages correctly to target channels

## Development Commands

```bash
# Validate CloudFormation template
aws cloudformation validate-template --template-body file://template.yaml

# Deploy stack
aws cloudformation deploy --template-file template.yaml --stack-name waf-notifier --parameter-overrides ParameterKey=Value

# Delete stack
aws cloudformation delete-stack --stack-name waf-notifier
```