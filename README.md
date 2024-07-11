# SAM

## 概要

サーバーレスアプリケーションを IaC で構築するためのフレームワーク。
CDK と同じく、最終的には Cloudformation のスタックとしてデプロイされる。

CLI (SAMCLI)とテンプレートファイルからなる。
テンプレートファイルに Lambda の設定が含まれている場合は、Lambda のコードも含まれる。

Teraform とも連携できる
CDK とはなし

Lambda はコードを含むので他のリソースとデプロイサイクルが違ってくる。
(基本的にインフラのデプロイよりコード変更のデプロイのほうが多いはず)

Lambda のみを SAM にしておいて、ほかのリソースは任意の IaC で作ることもできる。
Role や Log Group も事前に作って Export するか SSM に ARN を書き込んでおけば、SAM で読み込んで使える。

### sam init

`sam init`で良く使われる構成を自動生成できるが、自分でテンプレートファイルを作成して SAMCLI に読み込ませることも可能。
`sam init`コマンドで何かリソースが作られることはない

`sam init`で作られるテンプレートファイルのベースは[ここ](https://github.com/aws/aws-sam-cli-app-templates)にある

### テンプレートファイル

ほぼ Cloudformation のテンプレートと同じ。[違いは Doc 参照](https://docs.aws.amazon.com/ja_jp/serverless-application-model/latest/developerguide/sam-specification-template-anatomy.html)

AWS::Serverless::xxx をリソースとして使えるのが違い。
CDK の L2 コンストラクトと同じく、良く使われる構成に自動変換してくれるので、少ない記述でかける

ただし、明示せずともリソースを作ってくれる分、変更セットは注意して確認した方がいい。
また、思ったよりデプロイに権限が必要だったりする。

[リソースごとの注意点](https://docs.aws.amazon.com/ja_jp/serverless-application-model/latest/developerguide/sam-specification-generated-resources.html)

### sam deploy

テンプレートファイルを Cloudformation テンプレートファイルに変換して、CFn のスタックとしてデプロイするコマンド
Clouformation を実行し、リソースを作成するので、コマンドを実行するときに事前に設定する AWS Credential は十分な権限が必要

後述するが、`sam package`コマンドが実施することを内包している

### samconfig.toml

SAMCLI の設定ファイル。デプロイ時のスタック名などを記入して、毎回入力しなくて済むようにできる。

`sam deploy`時に「--guided」オプションをつければインタラクティブに samconfig.toml を生成できる

### sam package

AWS::Serverless::Function をテンプレートファイルに含んでいる場合に Lambda のソースコードをビルド・ZIP 化・S3 へアップロードを行ってくれる。
この時、アップロード先の S3 を使うように変更されたテンプレートファイルが出力される。

`sam deploy`はこの変更後テンプレートファイルから Cloudformation テンプレートファイルを作る

### sam codepipeline

packaging から deploy までを CI/CD として実行してくれるパイプラインを作ることができる。
これ自体は Cloudformation のテンプレートを自動で作ってくれる補助機能のようなもの。
CodeBuild 中で sam コマンドを使っているが、`sam pipelein init`や`sam pipeline bootstarap`で作られるリソースは sam とは関係がない

CodeBuild で使える AWS マネージドなイメージは基本 SAMCLI がインストールされている。
イメージの詳細は[ここ](https://docs.aws.amazon.com/ja_jp/serverless-application-model/latest/developerguide/sam-specification-generated-resources.html)を参照

CodeBuild イメージにインストールされている AWS CLI、AWSSAM を使用する場合はおそらく...CodeBuild に設定したサービスロールが権限として付与される [参考](https://dev.classmethod.jp/articles/lambda-codebuild-aws-cli/)

CodeBuild 時に必要な権限は bootstrap で生成される CFn テンプレートファイルが参考になりそう。

`sam package`時 (CodeBuild のログなど除いている)
S3 にアップロードする必要があるので、S3 系の権限

```
- "s3:GetObject"
- "s3:GetObjectVersion"
- "s3:PutObject"
```

`sam deploy`時
CFn の実行が必要なので、CFn 系の権限

[参考](https://github.com/aws/aws-sam-cli/blob/f791ac3d9c1aae5bb4fc9aacb0ecb29e1eba507f/samcli/lib/pipeline/bootstrap/stage_resources.yaml#L264-L330)

ただし、`sam deploy`のオプションとして、`--role-arn`オプションで CFn が使う権限も指定が必要
こちらはほぼ管理者権限と同等になっている[参考](https://github.com/aws/aws-sam-cli/blob/f791ac3d9c1aae5bb4fc9aacb0ecb29e1eba507f/samcli/lib/pipeline/bootstrap/stage_resources.yaml#L89-L111)

SAM テンプレートファイルには CFn と同じくほぼすべてのリソースを定義できるので、ほぼすべてのリソースを作れる権限が必要になってしまうためこうなっていると思われる。
ただし、bootstarap で作らない場合でテンプレートファイルに記載するリソースが限られる場合は、自分で IAM Role を作ってオプションに指定した方が、権限を絞れる。

以下 2 点は不明

- なぜ assume-role.sh で権限を切り替えているか
- なぜ CodePipeline 実行用の IAM User を作るのか

---

# 以下自動生成ドキュメント

---

# sam-app

This project contains source code and supporting files for a serverless application that you can deploy with the AWS Serverless Application Model (AWS SAM) command line interface (CLI). It includes the following files and folders:

- `src` - Code for the application's Lambda function.
- `events` - Invocation events that you can use to invoke the function.
- `__tests__` - Unit tests for the application code.
- `template.yaml` - A template that defines the application's AWS resources.

Resources for this project are defined in the `template.yaml` file in this project. You can update the template to add AWS resources through the same deployment process that updates your application code.

If you prefer to use an integrated development environment (IDE) to build and test your application, you can use the AWS Toolkit.  
The AWS Toolkit is an open-source plugin for popular IDEs that uses the AWS SAM CLI to build and deploy serverless applications on AWS. The AWS Toolkit also adds step-through debugging for Lambda function code.

To get started, see the following:

- [CLion](https://docs.aws.amazon.com/toolkit-for-jetbrains/latest/userguide/welcome.html)
- [GoLand](https://docs.aws.amazon.com/toolkit-for-jetbrains/latest/userguide/welcome.html)
- [IntelliJ](https://docs.aws.amazon.com/toolkit-for-jetbrains/latest/userguide/welcome.html)
- [WebStorm](https://docs.aws.amazon.com/toolkit-for-jetbrains/latest/userguide/welcome.html)
- [Rider](https://docs.aws.amazon.com/toolkit-for-jetbrains/latest/userguide/welcome.html)
- [PhpStorm](https://docs.aws.amazon.com/toolkit-for-jetbrains/latest/userguide/welcome.html)
- [PyCharm](https://docs.aws.amazon.com/toolkit-for-jetbrains/latest/userguide/welcome.html)
- [RubyMine](https://docs.aws.amazon.com/toolkit-for-jetbrains/latest/userguide/welcome.html)
- [DataGrip](https://docs.aws.amazon.com/toolkit-for-jetbrains/latest/userguide/welcome.html)
- [VS Code](https://docs.aws.amazon.com/toolkit-for-vscode/latest/userguide/welcome.html)
- [Visual Studio](https://docs.aws.amazon.com/toolkit-for-visual-studio/latest/user-guide/welcome.html)

## Deploy the sample application

The AWS SAM CLI is an extension of the AWS CLI that adds functionality for building and testing Lambda applications. It uses Docker to run your functions in an Amazon Linux environment that matches Lambda. It can also emulate your application's build environment and API.

To use the AWS SAM CLI, you need the following tools:

- AWS SAM CLI - [Install the AWS SAM CLI](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/serverless-sam-cli-install.html).
- Node.js - [Install Node.js 20](https://nodejs.org/en/), including the npm package management tool.
- Docker - [Install Docker community edition](https://hub.docker.com/search/?type=edition&offering=community).

To build and deploy your application for the first time, run the following in your shell:

```bash
sam build
sam deploy --guided
```

The first command will build the source of your application. The second command will package and deploy your application to AWS, with a series of prompts:

- **Stack Name**: The name of the stack to deploy to CloudFormation. This should be unique to your account and region, and a good starting point would be something matching your project name.
- **AWS Region**: The AWS region you want to deploy your app to.
- **Confirm changes before deploy**: If set to yes, any change sets will be shown to you before execution for manual review. If set to no, the AWS SAM CLI will automatically deploy application changes.
- **Allow SAM CLI IAM role creation**: Many AWS SAM templates, including this example, create AWS IAM roles required for the AWS Lambda function(s) included to access AWS services. By default, these are scoped down to minimum required permissions. To deploy an AWS CloudFormation stack which creates or modifies IAM roles, the `CAPABILITY_IAM` value for `capabilities` must be provided. If permission isn't provided through this prompt, to deploy this example you must explicitly pass `--capabilities CAPABILITY_IAM` to the `sam deploy` command.
- **Save arguments to samconfig.toml**: If set to yes, your choices will be saved to a configuration file inside the project, so that in the future you can just re-run `sam deploy` without parameters to deploy changes to your application.

## Use the AWS SAM CLI to build and test locally

Build your application by using the `sam build` command.

```bash
my-application$ sam build
```

The AWS SAM CLI installs dependencies that are defined in `package.json`, creates a deployment package, and saves it in the `.aws-sam/build` folder.

Test a single function by invoking it directly with a test event. An event is a JSON document that represents the input that the function receives from the event source. Test events are included in the `events` folder in this project.

Run functions locally and invoke them with the `sam local invoke` command.

```bash
my-application$ sam local invoke ScheduledEventLogger --event events/event-cloudwatch-event.json
```

## Add a resource to your application

The application template uses AWS SAM to define application resources. AWS SAM is an extension of AWS CloudFormation with a simpler syntax for configuring common serverless application resources, such as functions, triggers, and APIs. For resources that aren't included in the [AWS SAM specification](https://github.com/awslabs/serverless-application-model/blob/master/versions/2016-10-31.md), you can use the standard [AWS CloudFormation resource types](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-template-resource-type-ref.html).

Update `template.yaml` to add a dead-letter queue to your application. In the **Resources** section, add a resource named **MyQueue** with the type **AWS::SQS::Queue**. Then add a property to the **AWS::Serverless::Function** resource named **DeadLetterQueue** that targets the queue's Amazon Resource Name (ARN), and a policy that grants the function permission to access the queue.

```
Resources:
  MyQueue:
    Type: AWS::SQS::Queue
  ScheduledEventLogger:
    Type: AWS::Serverless::Function
    Properties:
      Handler: src/handlers/scheduled-event-logger.scheduledEventLogger
      Runtime: nodejs20.x
      DeadLetterQueue:
        Type: SQS
        TargetArn: !GetAtt MyQueue.Arn
      Policies:
        - SQSSendMessagePolicy:
            QueueName: !GetAtt MyQueue.QueueName
```

The dead-letter queue is a location for Lambda to send events that could not be processed. It's only used if you invoke your function asynchronously, but it's useful here to show how you can modify your application's resources and function configuration.

Deploy the updated application.

```bash
my-application$ sam deploy
```

Open the [**Applications**](https://console.aws.amazon.com/lambda/home#/applications) page of the Lambda console, and choose your application. When the deployment completes, view the application resources on the **Overview** tab to see the new resource. Then, choose the function to see the updated configuration that specifies the dead-letter queue.

## Fetch, tail, and filter Lambda function logs

To simplify troubleshooting, the AWS SAM CLI has a command called `sam logs`. `sam logs` lets you fetch logs that are generated by your Lambda function from the command line. In addition to printing the logs on the terminal, this command has several nifty features to help you quickly find the bug.

**NOTE:** This command works for all Lambda functions, not just the ones you deploy using AWS SAM.

```bash
my-application$ sam logs -n ScheduledEventLogger --stack-name sam-app --tail
```

**NOTE:** This uses the logical name of the function within the stack. This is the correct name to use when searching logs inside an AWS Lambda function within a CloudFormation stack, even if the deployed function name varies due to CloudFormation's unique resource name generation.

You can find more information and examples about filtering Lambda function logs in the [AWS SAM CLI documentation](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/serverless-sam-cli-logging.html).

## Unit tests

Tests are defined in the `__tests__` folder in this project. Use `npm` to install the [Jest test framework](https://jestjs.io/) and run unit tests.

```bash
my-application$ npm install
my-application$ npm run test
```

## Cleanup

To delete the sample application that you created, use the AWS CLI. Assuming you used your project name for the stack name, you can run the following:

```bash
sam delete --stack-name sam-app
```

## Resources

For an introduction to the AWS SAM specification, the AWS SAM CLI, and serverless application concepts, see the [AWS SAM Developer Guide](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/what-is-sam.html).

Next, you can use the AWS Serverless Application Repository to deploy ready-to-use apps that go beyond Hello World samples and learn how authors developed their applications. For more information, see the [AWS Serverless Application Repository main page](https://aws.amazon.com/serverless/serverlessrepo/) and the [AWS Serverless Application Repository Developer Guide](https://docs.aws.amazon.com/serverlessrepo/latest/devguide/what-is-serverlessrepo.html).
