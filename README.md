# AWS Developer Concepts

## IAM
-  Iam policy: effect, principle, action, resource, to give permission to users and groups
-  inline policy(on principal creation), customer managed policy(create policy and attach to principal), aws managed policy(attach automatically by aws to principal on creation)
-  passrole: give permission to a user to assign a role to a service
-  trustpolicy: a role can be assigned only to its trusted resources
-  Password policy 
-  Mfa
-  Iam role: to give permission to services
-  Access advisor, credential report

## EC2: 
-  user data
-  security groups: only allow rules
-  purchasing options: on demand, saving, reserved, spot, dedicated
-  storage: 
	- ebs(persistant,attach to 1 instance(except io1,io2),1 az (move with snapshots),gp2,gp3,io1,io2,st1,sc1)
	- efs(scalable, multi instance, expensive)
	- instance store (ephemeral, default, high performance)

## ELB
-  (CLB,ALB,NLB,GLB(give traffic to ids,ips,firewall)),target group, health check at target group
-  alb routing based on path,hostname,query strings, header
-  alb target groups: ec2, ecs task, lambda, ip addresses(priv),
-  ip and port of clients on X-Forwarded-For and X-Forwarded-Proto
-  nlb tg: alb, ec2, ip(priv)
-  sticky session using cookie (application cookie, custom cookie, duration based cookie)
-  cross zone load balancing
-  https listener: manage certs with acm, manage multiple certs in 1 place with sni
-  connection draining(deregistration delay) for unhealthy instances

## ASG 
-  launch template, scaling policies,
-  dynamic scaling:target tracking, simple scaling, scheduled
-  cooldown time
-  instance refresh (for update instances), min healthy instances, warm-up time

## RDS 
-  storage auto scaling,
-  read replicas: up to 15, eventually consistent, cross az and region support, can be promoted to db
-  multi az rds: master-standby, for DR, no manual intervention, can be enabled with zero down time
-  rds proxy to reduce number of connections, accessable from vpc(not publicly)

## Aurora
-  for postgres and mysql, HA native, faster than normal ones,storage auto scale, write on master with multi read replicas,cross region is supported, writer endpoint and reader endpoint,
-  global aurora: for cross region replication,1 primary region(read/write),secondary regions(read only)

## Elasticash
-  managed redis(replication) or memcashd(sharding), 
-  make app stateless(store sessions then apps instances read from there=>prevent overloaded instances), 
-  to reduce read from db(only on cache miss read from db)
-  cache eviction using ttl
-  lazy loading: for read,1.cache miss=> 2.read from db,3.write to cache
-  write through: for write, 1.write to db, 2.write to cache
-  replication: cluster mode disabled(1 primary node(read+write)+read replicas,improve read,async replication), clustermode enabled(sharding data.improve write)
-  clients connect to reader/writer endpoint, their requst will be loadbalanced between elasticache nodes
-  auto scaling for replicas(only with cluster mode enabled) with cloudwatch metrics and alarm

## Route53:
-  records contain: domain, value, record type, routing policy, ttl
-  record types: a,aaaa,cname(map hostname to another hostname), ns
-  cant create cname for zone apex(example.com)
-  hosted zone: container for records, public(from internet) and private(from ec2 instances)
-  alias: map hostname to aws resource, automatically notices ip changes, no ttl, record type= A record
-  alias targets: elb,cloudfront, beanstalk,s3,api gw, NOT ec2 dns name
-  route policies: simple(no health check),weighted, failover(active-passive/primary-secondary), latency based, geolocation, multivalue, geoproximity(based on bias, config using traffic flow), client's ip based
-  hc using cloudwatch alarm or route53 hc

## VPC
-  public and private subnets, internet gw, nat gw(1 nat gw in each AZ for HA)
-  nacl: allow and deny traffic from and to a subnet, return traffic should be allowed
-  sg: only allow traffic from and to an ec2 instance, return traffic is allowed by default
-  traffic flow logs: capture traffic of vpc, subnet, elb, rds, ..., and gives it to s3,cloudwatch logs, kinesis data firehose
-  vpc peering: not transitive(no vpc peering through other vpc s)
-  vpc endpoint(eni): connect instance (inside subnets) to aws services
-  vpc endpoint gw: connect instance (inside subnets) to s3 and dynamodb

## S3
-  object=file, bucket=directory, key= prefix(folder)+object name, max 5G upload (multipart upload)
-  security: user or ec2(iam policy or iam role), resource-based(bucket policy(for public access, cross account acccss, enforce in transit enc), object acl, bucket acl), 
-  the union of iam policy(attached to user,role,group) and bucket policy(attached to s3 bucket) will evaluate the access
-  dynamic policy: ex assign each user a /home/<user> folder in an S3 bucket (use /home/${aws:username} in user's iam policy)
-  versioning: enabled at bucket level, for same key overwrite, delete marker instead of permanent delete.
-  replication: crr(cross region),srr(same region), enabled versioning is needed, async
-  only for new files after enabling replication, for existing files, use s3 batch replications,
-  no auto chaining in replication
-  storage classes: standard(general purpose, infrequent access), one zone, glacier(instant retrival,flexible retrival, deep archive),intelligent tiering
-  move between classes manually or using s3 lifecycle rules(for certain prefix or object tags)
-  event notifications: on any action on s3, can be sent to sns,sqs,lambda,eventbridge. s3 needs enough permission (on sns,sqs,lambda resource policy) to send notifications
-  transfer acceleration to increase transfer speed
-  s3 byte range fetch: request specific byte ranges
-  s3 select: retrieve less data by server side filtering using sql
-  object tags: store tags on a db and search on it(search by tags, are not possible on s3 directly)
-  security: sse,sse-kms,sse-c,client side enc, in transit enc using tls
-  CORS: cross origin(domain+protocol+port), client->origin->host
-  mfa delete: for permanently delete objects or disable versioning,
-  access logs(authorized or denied) of a bucket to another bucket(in same region)
-  presigned url to give temp access to an object to a person
-  access point: a dns name+ its own access point policy. to have better security or change data(s3->lambda->ap->client)

## IMDS
-  instance metadata, info about ec2 can be retrieved by url(http://169.254.169.254/latest/meta-data)
-  mfa with cli: using "sts get-session-token" api call
-  service quotas, throttling exception, exponential backoff
-  sdk and cli credentials: if principal is on aws, use role to give permission, if not, use env vars to store access key and secret of an allowed account
-  requests from sdk and cli are signed. to send direct http req to aws, we should sign it using sig v4

## CloudFront
-  cdn, improve read performance,cache content on global edges, for static contents(ttl= around 1 day)
-  read cache using cache key(default=hostname+resource part in url, but we can change it)
-  origin req policy: which headers should be forwarded to origin by cloudfront to get the content
-  cache behaviour: origins(s3,lb) can be different for each path, and routing to right origin is done by cache behaviour
-  vs s3 replication: for dynamic contents on few regions
-  geo restriction
-  signed url/cookie for premium contents. vs s3 presigned req: s3 presigned uses iam keys of issuer
-  origin group to handle failover (primary and secondary origin)
-  field level encryption(fields of http req are encrypted in cloudfront and origins(lb)(only destination can read it))
-  send logs to kinesis data stream.

## ECS
-  ecs tasks on ecs cluster
-  ecs launch type: ec2(+ecs agent on ec2 to register ec2 on ecs cluster), fargate
-  ecr: like docker hub,controlled access by iam, public/private
-  ec2 instance profile used by ecs agent for: send cont log to cloudwatch, access ecr, ssm parameter store, secret manager. same for all tasks on that ec2
-  ecs task role: diffrent role for different task definitions
-  elb integration, persistant data using efs
-  ecs task auto scaling based on cpu,ram,req count. (!= ec2 scaling)
-  scaling models: target traking(cloudwatch metrics), step scaling(cloudwatch alarm), scheduled.
-  capacity provider: scale task first, if needed, scale ec2
-  rolling update(min, max healthy tasks)
-  ecs task can be called by eventbridge
-  notify admin when task stopped(ecs->eventbridge->sns->email to admin)
-  task definition:image,port,env vars,ram,cpu,iam role
-  lb integragion, can find host port dynamicly(in task definition 0(random host port):80(cont port), allow all ports from lb on ec2 sg)
-  env vars sources: ssm parameter store, secret manager, hardcoded in code itself
-  placement constraints(cpu,ram,distinct instance,member of(on ec2 instances with specific properties)), placement strategies(binpack(least available resource),random,spread)

## EKS 
-  launch type: ec2, fargate,
-  collect logs and metrics using cloudwatch container insight
-  storage supports ebs and efs, leverages csi driver

## Beanstalk
-  environment tiers(web server,worker), deployment mode(single instance, HA with LB)
-  update options: All at once,Rolling,Rolling with additional batches,Immutable(new temp asg),Blue Green(new env,route53 to canary,finally swap url), Traffic Splitting(canary)
-  create code, zip it, upload zip file to deploy
-  lifecycle policy(max 1000 versions are allowed=>delete old versions)
-  env config can be put in .ebextensions/<filename>.config (yaml/json) in code's zip file
-  relies in cloudformation
-  env can be cloned (rds data will not copppied to the clone env), usefull for test. to change lb type we can not use clone option(create new env manually)

## CloudFormation
-  create template->upload to s3->use it in cloud formation stack
-  template can be created using cloudformation designer or yaml file
-  parts of template:resources, parameters(dynamic values),mappings(static values),outputs,conditionals(controll creation of resources and outputs),metadata,refrences,functions
-  !Ref(for parameters or resources(returns resource id)),!FindInMap(for maps),!ImportValue(for outputs),!Equal(for conditions),!GetAttr(to get properties of a resource),!Sub(to substitute variable name with its value),
-  pseudo parameters(default aws parameters,(region,stack name,...))
-  rollback for creation or update fails
-  stack notifications can be sent to sns
-  change set:shows changes before update the stack
-  nested stacks(for load balacer,security groups,...)
-  cross stacks(when lifecycle of stacks are different(use outputs and !ImportValue))
-  stackset: create,update,delete stacks across multiple accounts or regions. update stackset=update stacks instances in all accounts
-  drift detection: detect manual changes on stack
-  stack policy: allow update of specific resource on stack(protect unintentional resource update during stack update). when enabled, default is deny,when is disabled, default is allow

## Messaging
-  sns,sqs,kinesis
-  sqs:unlimited throughput,max msg size=256KB, max retention=14 days, retrieve up to 10 messages at a time, producer=sdk,consumer=ec2,lambda,server. consumer process and deletes messages.
-  encryption:in flight(https), at rest(kms), client side
-  access control by iam policy
-  sqs access policy: allow other services to write on sqs + cross account access
-  message visibility: default=30sec. when a consumer poll a msg, how much it will be invisible to other consumers. how long takes processing a message. if proccess takes more time, it will be processed twice(unless consumer notify sqs using change message visibility)
-  fifo queue:exactly once, remove duplicates, ordered processing(within message group,not across message groups)
-  dlq: in case of failure loop(msg come back to queue multiple times), after a threshold, message will be sent to dlq. after solving the problem, it comes back to sqs using redrive
-  long polling: in case of empty sqs queue, holds the polling request, once a msg produced, responds to consumer(no empty responces) => less api call => preferred
-  extended client: for large messages, put message on s3, put its address to sqs,consumer retrieve msg from s3 by the address
-  deduplication: prevent duplicate messages checking the hash of the msg
-  message group: group messages using grouping id, max 1 consumer per group id

-  sns: send 1 msg to many receivers (subscribers to a topic), unlimited number of subscribers per topic. 
-  pubblishers: cloudwatch alarm,lambda,asg,s3,cloudformation,...
-  subscribers: sqs,lambda,kinesis fire hose,sms,email
-  pubblish using sdk
-  "encryption, access controll, access policy, fifo" like sqs
-  sns+sqs fan out: sns send to multi sqs=> multi consumers poll messages. benefit: support delayed processing for sns messages. sns should be allowed on sqs access policy.
-  fan out usecases: send s3 event notification to multiple queues,
-  fan out+ordering+deduplication=sns fifo+sqs fifo fan out
-  message filtering: to give specific messages(based on content of msg) of a topic to a subscriber

-  kinesis: real time data stream processing
-  data stream: capture data
-  firehose: gives data to other services
-  data analytics: analyse data stream using sql(usecase:dashboard)

-  data stream:
-  producers send records(partition key(id of producer)+data blob(max 1MB)+seq no.) through shards to consumers
-  shards: max 1MBps in, max 2MBps out (enhanced:max 2MBps out per consumer), choose shard based on hash of partitoin key(same partition key always passes through same shard)
-  producers: sdk, kpl(kinesis producer library),kinesis agent
-  consumers: lambda, sdk,firehose,data analytics,kcl(kinesis client library, multiple kcl s can not read from 1 shard, 1 kcl can read from multiple shards)
-  capacity modes: provisioned, on demand
-  "encryption, access controll, access policy" like sqs
-  ordered at shard level
-  shard splitting: to devide hot shards
-  merging shard

-  firehose
-  producers: data stream, cloudwatch logs, eventbridge,sdk
-  destinations: aws(s3,opensearch) and non aws services
-  can modify data using lambda before sending
-  near real time: 60 seconds time slots, or wait until batch of msg s reaches to 1MB

## Monitoring
-  cloudwatch(metric,logs,events,alarm),xray(trace interactions btw miroservices), cloudtrail(audit api calls to aws resources)
-  cloudwatch logs: metric(like cpuutilization), dimension(attribute of metric like instance id,env,...)
-  ec2 metrics: every 5 minutes(detailed monitoring=1min), memory needs custom metrics
-  custom metrics: using putmetricdata, every 1 minutes(high resolution=up to 1 sec)
-  log group: represent an app,
-  log stream: log files, for instances of an app
-  logs can be sent to : s3(export, needed up to 12 hours),data stream, firehose,lambda
-  log sources: cloudwatch unified agent,sdk,ecs,lambda,vpc flow logs,api gw, cloud trail,route 53,beanstalk
-  logs insight: search in logs, query engine
-  logs subscription: send logs in realtime to data stream, firehose,lambda. filter logs to be sent using subscription filter
-  log aggregation: multi region or account to-> multi subscription filter->data stream->fire hose->s3 (near real time)
-  ec2 logs needs cloudwatch unified agent to be sent to cloudwatch(not be sent by default)
-  cloudwatch unified agent collects:cpu,disk,ram,net,processes,swap space
-  metric filter: is done after sending logs to cloudwatch: agent->cw->metric filter->cw alarm->email. ex1 find specific ip in logs, ex2 count errors
-  logs can be encrypted using kms keys, encryption at log group level

-  cloudwatch alarms: for any metric(for 1 metric), for custom metrics up to every 10 sec
-  targets:ec2(stop,terminate,reboot, recover(same ip,metadata,placement group)), trigger auto scaling in asg, sns to send notifications
-  composit alarm: consider multi metrics(using AND or OR multiple alarm)
-  syntetics canary: on schedule performs some predefined ui test, if failed: cw alarm->lambda->change route53 to point to another instance

-  eventbridge: trigger event on schedule or on specific action, to trigger something
-  source: ec2,s3 event,cloudtrail,codebuild, or on schedule
-  destinations: almost every where(lamda,ecs task,sqs,sns,data stream, stepfunc,codepipeline,codebuild,ssm,ec2 action)
-  resource based policy: to give access to other accounts or regions to read from or write to event bus (used in aggregation)

-  xray compatibilities: lambda,ec2,ecs,beanstalk,api gw,elb
-  to use: import xray sdk in code, run xray daemon(on container on ec2 or as a sidecar container),give proper iam role to ec2
-  trace: end to end trace (consists of segments)
-  distro: opensource version of xray (based on open telemetry)

-  cloud trail:
-  get history of api calls (by console,sdk,cli,aws services) in aws account 
-  can be sent to cloudwatch logs and s3
-  retention: max 90 days, store on s3 to save them, use athena to search and analyse
-  cloudtrail->eventbridge->sns to send email

## Lambda
-  syncronous invocation: resault is returned instantly. invoke by user(cli,sdk,api gw,alb) or service(cognito,step func)
-  invoke by alb:invocation by alb should be allowed in lambda resource policy. alb converts http req to json, lambda func should be in target group of alb
-  lambda event source mapping: get data (from data stream,fifo sqs,fifo sns,dynamo db streams in batches) and invoke lambda syncronously
-  asyncronous invocation: put invocations on an event queue, dlq for failed events. 
-  invoke by (s3(generate thumbnail),sns,eventbridge,ses(simple email service),cloud formation,codecommit,codepipeline)
-  lambda recieves event object(data to process) and context object(info about invocation) on invocations, accessable inside lambda
-  result of lambda func can be send to sqs,sns using destination (for async invok) or event source map
-  lambda execution role: give permission to lambda to access aws resources
-  lambda resource based policy: give permission to aws resources to access lambda
-  to give permission to users to access lambda, use iam policy and attach it to user
-  send logs to cloudwatch(should be allowd on execution role)
-  by default is run outside of our vpc, and can access internet
-  to run it on our vpc, it needs an eni(elastic network interface) (in this case it needs nat gw and internet gw to access to internet)
-  to increase cpu, increase ram
-  timeout default=3sec, max=15min
-  initialize db connection outside of handler(establish db connection once,not on every invokation)
-  if lambda func needs large files, use /temp dir, it will be kept there for some time. for large persistant data use s3
-  efs can be used as its storage(efs access point is needed, and lambda should be run in our vpc)
-  set reserved concurrency for functions to avoid throttling of other function due to throttling of another function
-  cold start: load code and run every thing which is outside of handler
-  provisioned concurrency: make some lambda funcs run before invokation (usefull when cold start is so long)
-  dependencies: zip dependencies with code and upload to lambda(if it is more than 50MB, upload it on s3 first)
-  create lambda with cloud formation: inline(define in cloud formation template) or using s3(upload code on s3 and reference its path in template)
-  can be run in container using lambda container image
-  version: deploy updates, are immutable, $LATEST
-  alias: poiner to version, can enable canary deployment (x% to v1, y% to v2)

## DynamoDB
-  nosql db, can scale horizontally, no join query
-  table,primary key(partition key(hash)+ optionaly sort key), item(row), attribute(col)
-  read/write capacity modes: provisioned(rcu(read capacity unit)/wcu(write capacity unit)), on demand
-  eventually consistent read/ strong consistent read
-  scan:return entire table,then filter
-  query:filter, then return(less rcu consumption)
-  conditional write: ex do the operation if the attribute exists, or optimistic locking(do if value=somthing)
-  local secondary index: alternative sort key(same partition key), define in table creation, up to 5 lsi
-  global secondary index: alternative primary key, speed up query on non key attributes
-  DAX: dynamo db accelerator, cache in memory => less rcu
-  db streams: sent modification on items to data stream or lambda
-  ttl: delete item on specific unix epoch
-  can be used as a store for session info(attach efs for shared access)
-  write sharding: when we have few partition keys, add prefix to partition keys, avoid hot partition
-  for large files, upload file on s3, add its address to dynamo db
-  global dynamo db: for multi region, active-active replication, read/write on any replication
-  PITR: point in time recovery, for last 35 days
-  s3 integration: can export data to s3(to perform data anlysis), can import data from s3

## API-GW
-  user auth with: iam role,cognito, custom authorizer
-  https with acm certificate manager
-  make deployment stages to apply changes
-  each stage has its own stage variables(to map stages with lambda alias, canary deployment)
-  integration types: aws_proxy(pass req to lambda), http_proxy(pass req to backend), http/aws(change data using mapping template before pass to func)
-  export and import api s using open api
-  can validate request before sending to backend
-  can cache api responces
-  cloudwatch metrics: count, integration latency, latency
-  security: auth with iam(for users and aws services), resource policy(for cross account access), auth with cognito, lambda athorizer(for custom token auth like jwt)
-  support websocket (push info to client)

## CI/CD
-  code pipeline:source(codecommit),build & test(codebuild,jenkins), deploy(codedeploy)
-  artifact of each stage will be saved on s3, and will be read from s3 as input of next stage

-  codecommit: like github
-  security: authentication(ssh key/https),authorization(iam policy to give permission to users to access to repo),cross account access(iam role and sts)

-  codebuild: build instructions on buildspec.yml (or insert manualy on console), put it in root folder of code
-  buildspec.yml:env(env vars, enter value or read value form parameter store or secret manager)
	- phases:install(dependencies),pre_build,build,post_puild. enter commands to execute in each phase
	- artifact: path to final artifact, will be uploaded to s3
	- cache: files to cache (dependencies) to S3 for future build speedup
	- secrets should be stored in ssm parameter store and secret manager, then reference them in code build as env var

-  codedeploy: describe how to deploy on appspec.yml file, can use hooks to verify deployment after each phase
-  need codedeploy agent on machines,should have permission to get artifact from s3
-  options for ec2: in place(AllAtOnce, HalfAtATime, OneAtATime, Custom), blue green(new asg)
-  options for lambda: linear,canary,AllAtOnce
-  options for ecs(only blue green): linear,canary,AllAtOnce (how to redirect alb from blue to green)
-  can deploy on ec2,lambda,ecs

-  codeartifact: to manage code dependencies. update a dependency on codeartifact->eventbridge->start codepipeline to re build and re deploy our app
-  resource policy: to give permission to other accounts to access codeartifact

-  codeguru: review and give recomendations on our code using machine learning
-  cloud9: ide on web(cloud)

## SAM
-  Serverless Application Model, like cloud formation, but just for serverless services, starts with "Transform:'AWS::Serverless..."
-  workflow: create template (yaml file)->sam build(transform to cloudformation)->sam package(zip and upload to s3)-> sam deploy(create cloudformation stack)
-  can create and test localy using sam cli(sam local)+aws toolkit (emulate lambda,api gw)
-  deploymentpreference in template: type(canary,linear,all at once),monitor alarms to do rollback, pre and post traffic shift hooks using code deploy under the hood

## CDK
-  cloud development kit, create cloud formation template using programing languages(javascript,python)->cdk synth(transform to cloud formation template in json)

## Cognito
-  to authenticate non aws users to api gw and alb using user pools(serverless db, local or from facebook and google)
-  alb does the auth=>simpler code in backend
-  identity pool: can be used to give access to non aws users to aws resources creating temp sts credentials after successfull login through user pool
	- create an iam role to define what can be accessed by users
	- to give user specific access use policy variables in iam role(ex: user can access to a path in s3 only if it starts with user's id)

## Step Function
-  state machine, to define workflow, use json, can be triggered by sdk and api gw and eventbridge
-  tasks:invoke lambda,run batch job,run ecs task and wait to finish,publish msg on sns sqs,insert itam in dynamo db
-  states:choice,failed,succeed,pass,wait,parallel
-  can retry failed tasks(if max retry reaches, catch will come up(send it to another state))

## AppSync (graphql)
-  get data from multiple types of db(nosql,sql,api) and give it to the app

-  amplify: create mobile apps and web apps (ui tools, data store, hosting)

-  sts: security token service, give limited and temporary access to AWS resources (up to 1h)(assumerole from another role)

-  directory servics: managed AD(AD on aws, trust with on premise AD), AD connector(redirect to on premise AD), simple AD(AD on aws, no trust relationship)

## KMS: key management service, 
-  key types: symetric(key not downloadable, enc/dec using kms api), asymetric(public key downloadable)
-  key types: aws owned,aws managed,customer managed
-  auto key rotation,
-  key policy: who can access the key
-  envelop enc: if size of secret is more than 4KB, kms send data key for enc and dec=> client side enc and dec using data key
-  s3 bucket key: create by kms, create data keys form that to enc/dec objects in s3=>less api call to kms=>avoid kms throttling

## SSM Parameter Store
-  key value, store in hierarchy approach(department/app/dev/db-url), for secrets and non secret vars
-  advanced parameters feature: not free,parameter policy,ttl(expiration date)

-  secret manager: same approach as parameter store, for secrets only, auto key rotation with lambda, replication through muliple regions
-  set "ManagedMasterUserPassword: true" in cloud formation in rds, to automatically create and manage db admin pass on secret manager

## Others
-  Nitro Enclave: to process high sensitive data on isolated vm s

-  ses:simple email service, send email using aws console,api,smtp

-  opensearch: query on any fields of dynamodb(not only primary key), search on cloudwatch logs(cw logs->subscription filter->firehose->opensearch)

-  athena: search on s3 using sql

-  msk: managed streaming for apache kafka, alternative for kinesis(for pub sub)

-  acm: certificate manager, auto renewal, integration with elb,api gw, cloudfront

-  macie: alert when it found sensitive data on s3

-  appconfig: change config of app without changing the code(change on parameter store)

-  cloudwatch evidently: route some users to new version of app

-  HSM: enc/dec with hardware