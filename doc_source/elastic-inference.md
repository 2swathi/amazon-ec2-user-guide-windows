# Amazon Elastic Inference<a name="elastic-inference"></a>

Amazon Elastic Inference is a resource you can attach to your Amazon EC2 instances to accelerate your deep learning \(DL\) inference workloads\. Amazon EI accelerators come in multiple sizes and are a cost\-effective method to build intelligent capabilities into applications running on Amazon EC2 instances\. 

Amazon EI accelerates operations defined by TensorFlow, Apache MXNet, and the Open Neural Network Exchange \(ONNX\) format on low\-cost, GPU\-based, DL inference accelerators\. Developers building a wide range of applications on Amazon EC2 instances with machine learning inference workloads can benefit from wider deployment through the reduction in cost that Amazon EI enables\. 

**Topics**
+ [Amazon EI Basics](#elastic-inference-basics)
+ [Working with Amazon EI](#working-with-ei)
+ [Using CloudWatch Metrics to Monitor Amazon EI](#cloudwatch-metrics-ei)
+ [Troubleshooting](#ei-troubleshooting)

## Amazon EI Basics<a name="elastic-inference-basics"></a>

When you configure an Amazon EC2 instance to launch with an Amazon EI accelerator, AWS finds available accelerator capacity and establishes a network connection between your instance and the accelerator\.

Amazon EI accelerators are available to all EC2 instance types\.

The following Amazon EI accelerator types are available\. You can attach any Amazon EI accelerator type to any instance type\.


| Accelerator Type | FP32 Throughput \(TFLOPS\) | FP16 Throughput \(TFLOPS\) | Memory \(GB\) | 
| --- | --- | --- | --- | 
| eia1\.medium | 1 | 8 | 1 | 
| eia1\.large | 2 | 16 | 2 | 
| eia1\.xlarge | 4 | 32 | 4 | 

An Amazon EI accelerator is not part of the hardware that makes up your instance\. Instead, the accelerator is attached through the network using an AWS PrivateLink endpoint service\. The endpoint service routes traffic from your instance to the Amazon EI accelerator configured with your instance\.

Before you launch an instance with an Amazon EI accelerator, you must create an AWS PrivateLink endpoint service\. Each Availability Zone requires only a single endpoint service to connect instances with Amazon EI accelerators\. For more information, see [ VPC Endpoint Services \(AWS PrivateLink\)](https://docs.aws.amazon.com/vpc/latest/userguide/endpoint-service.html)\.

![\[An Amazon EI accelerator attached to an EC2 instance.\]](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/images/elastic-inference.png)

You can use Amazon Elastic Inference enabled TensorFlow Serving or Apache MXNet libraries to load models and make inference calls\. The modified versions of these libraries automatically detect the presence of Amazon EI accelerators, optimally distribute the model operations between the Amazon EI accelerator and the CPU of the instance, and securely control access to your accelerators using IAM policies\. The [AWS Deep Learning AMIs](https://aws.amazon.com//blogs/machine-learning/get-started-with-deep-learning-using-the-aws-deep-learning-ami) include the latest releases of Amazon Elastic Inference enabled TensorFlow Serving and MXNet\. If you are using custom AMIs or container images, you can download and install the required [Amazon Elastic Inference TensorFlow Serving](https://s3.console.aws.amazon.com/s3/buckets/amazonei-tensorflow/) and [Amazon Elastic Inference Apache MXNet](https://s3.console.aws.amazon.com/s3/buckets/amazonei-apachemxnet/) libraries from Amazon S3\. 

**Note**  
An Amazon EI accelerator is not visible or accessible through the device manager of your instance\.

The Amazon EI accelerator network traffic uses the HTTPS protocol \(TCP port 443\)\. Ensure that the security group for your instance and for your AWS PrivateLink endpoint service allows for this\. For more information, see [Configuring Your Security Groups for Amazon EI](working-with-ei.md#ei-security)\. 

### Pricing for Amazon EI<a name="elastic-inference-pricing"></a>

You are charged for each second that an Amazon EI accelerator is attached to an instance in the `running` state\. You are not charged for an accelerator attached to an instance that is in the `pending`, `stopping`, `stopped`, `shutting-down`, or `terminated` state\. You are also not charged when an Amazon EI accelerator is in the `unknown` or `impaired` state\.

You do not incur AWS PrivateLink charges for VPC endpoints to the Amazon EI service when you have accelerators provisioned in the subnet\.

### Amazon EI Considerations<a name="elastic-inference-limitations"></a>

Before you start using Amazon EI accelerators, be aware of the following limitations:
+ You can attach one Amazon EI accelerator to an instance at a time, and only during instance launch\.
+ You cannot share an Amazon EI accelerator between instances\.
+ You cannot detach an Amazon EI accelerator from an instance or transfer it to another instance\. If you no longer require an Amazon EI accelerator, you must terminate your instance\. To change the Amazon EI accelerator type, create an AMI from your instance, terminate the instance, and launch a new instance with a different Amazon EI accelerator specification\. 
+ Currently, only the Amazon Elastic Inference enhanced MXNet and Amazon Elastic Inference enhanced TensorFlow Serving libraries can make inference calls to Amazon EI accelerators\. 
+ Amazon EI accelerators can only be attached to instances in a VPC\.
+ Pricing for Amazon EI accelerators is available at On\-Demand rates only\. You can attach an accelerator to a Reserved Instance, Scheduled Reserved Instance, or Spot Instance\. However, the On\-Demand price for the Amazon EI accelerator applies\. You cannot reserve or schedule Amazon EI accelerator capacity\. 

### Choosing an Instance and Accelerator Type for Your Model<a name="choosing-types-eia"></a>

Demands on CPU compute resources, CPU memory, GPU\-based acceleration, and GPU memory vary significantly between different types of deep learning models\. The latency and throughput requirements of the application also determine the amount of instance compute and Amazon EI acceleration you need\. Consider the following when you choose an instance and accelerator type combination for your model:
+ Before you evaluate the right combination of resources for your model or application, you should determine the target latency and throughput needs for your overall application stack, as well as any constraints you may have\. For example, if your application needs to respond within 300 milliseconds \(ms\), and data retrieval \(including any authentication\) and pre\-processing takes 200ms, you have a 100ms window to work with for the inference request\. Using this analysis, you can determine the lowest cost infrastructure combination that meets these targets\. 
+ Start with a reasonably small combination of resources\. For example, a `c5.xlarge` instance type along with an `eia1.medium` accelerator type\. This combination has been tested to work well for various computer vision workloads \(including a large version of ResNet: ResNet\-200\) and give comparable or better performance than a `p2.xlarge` instance\. You can then size up on the instance or accelerator type depending your latency targets\. 
+ Since Amazon EI accelerators are attached over the network, input/output data transfer between instance and accelerator also adds to inferencing latency\. Using a larger size for either or both instance and accelerator may reduce data transfer time, and therefore reduce overall inferencing latency\.
+ If you load multiple models to your accelerator \(or, the same model from multiple application processes on the instance\), you may need a larger accelerator size for both the compute and memory needs on the accelerator\.
+ You can convert your model to mixed precision, which utilizes the higher FP16 TFLOPS of Amazon EI \(for a given size\), to provide lower latency and higher performance\. 

### Using Amazon Elastic Inference with EC2 Auto Scaling<a name="ei-autoscaling"></a>

When you create an Auto Scaling group, you can specify the information required to configure the Amazon EC2 instances, including Amazon EI accelerators\. To configure Auto Scaling instances with Amazon EI accelerators, you can specify a launch template with your instance configuration, along with the Amazon EI accelerator type\.

# Working with Amazon EI<a name="working-with-ei"></a>

To launch an instance and associate it with an Amazon EI accelerator, you must first configure your security groups and AWS PrivateLink endpoint services\. Then, you must configure an instance role with the Amazon EI policy\. 

**Topics**
+ [Configuring Your Security Groups for Amazon EI](#ei-security)
+ [Configuring AWS PrivateLink Endpoint Services](#eia-privatelink)
+ [Configuring an Instance Role with an Amazon EI Policy](#ei-role-policy)
+ [Launching an Instance with Amazon EI](#eia-launch)
+ [Using TensorFlow Models with Amazon EI](#ei-tensorflow)
+ [Using MXNet Models with Amazon EI](#ei-mxnet)
+ [Using ONNX Models with Amazon EI](#ei-ONNX)

## Configuring Your Security Groups for Amazon EI<a name="ei-security"></a>

You need two security groups: one for inbound and outbound traffic for the new Amazon EI VPC endpoint and another for outbound traffic for the associated EC2 instances that you launch\.

### Configure Your Security Groups for Amazon EI<a name="ei-security-group"></a>

**To configure a security group for an Amazon EI accelerator \(console\)**

1. Open the Amazon VPC console at [https://console\.aws\.amazon\.com/vpc/](https://console.aws.amazon.com/vpc/)\.

1. In the left navigation pane, choose **Security**, **Security Groups**, **Create a Security Group**\.

1. Under **Create Security Group**, enter field values and choose **Yes, Create**\. 

1. Select the box next to your security group and choose **Inbound Rules**\. 

1. To allow traffic from only port 443 from any source, or the security group to which you plan to associate your instance, for **Type**, select **HTTPS**\. 

1. Choose **Save**, **Outbound Rules**\. 

1. To allow traffic for port 443 to any destination, for **Type**, select **HTTPS**\.

1. Choose **Save**, **Outbound Rules**\.

1.  Add an outbound rule that either restricts traffic to the endpoint security group that you created in the previous step or that allows traffic to HTTPS \(TCP port 443\) to any destination\.

1. Choose **Save**\.

**To configure a security group for an Amazon EI accelerator \(AWS CLI\)**

1.  Create a security group using the `create-security-group` command: 

   ```
   aws ec2 create-security-group
   --description insert a description for the security group
   --group-name assign a name for the security group
   [--vpc-id enter the VPC ID]
   ```

1.  Create an inbound rule using the `authorize-security-group-ingress` command: 

   ```
   aws ec2 authorize-security-group-ingress --group-id insert the security group ID --group-name insert the name of the security group --protocol https 
   --port 443
   ```

1.  Use the `authorize-security-group-egress` command to create an outbound rule: 

   ```
   aws ec2 authorize-security-group-egress --group-id insert the security group ID --group-name insert the name of the security group --protocol https --cidr 0.0.0.0/0
   ```

1. Create a security group using the `create-security-group` command: 

   ```
   aws ec2 create-security-group
   --description insert a description for the security group
   --group-name assign a name for the security group
   [--vpc-id enter the VPC ID]
   ```

## Configuring AWS PrivateLink Endpoint Services<a name="eia-privatelink"></a>

Amazon EI uses [VPC endpoints](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-endpoints.html) to privately connect the instance in your VPC with their associated Amazon EI accelerator\. You must create a VPC endpoint for Amazon EI before you launch instances with accelerators\. This needs to be done just one time per VPC\. For more information, see [Interface VPC Endpoints \(AWS PrivateLink\)](https://docs.aws.amazon.com/vpc/latest/userguide/vpce-interface.html)\.

**To configure an AWS PrivateLink endpoint service \(console\)**

1. Open the Amazon VPC console at [https://console\.aws\.amazon\.com/vpc/](https://console.aws.amazon.com/vpc/)\.

1. In the left navigation pane, choose **Endpoints**, **Create Endpoint**\.

1. For **Service category**, choose **Find service by name**\.

1. For **Service Name**, select **com\.amazonaws\.us\-west\-2\.elastic\-inference\.runtime**\. 

1. For **Subnets**, select one or more Availability Zones where the endpoint should be created\. Where you plan to launch instances with accelerators, you must select subnets for the Availability Zone\. 

1. Enable the private DNS name and enter the security group for your endpoint\. Choose **Create endpoint**\. Note the VPC endpoint ID for later\. 

**To configure a AWS PrivateLink endpoint service \(AWS CLI\)**
+ Use the `create-vpc-endpoint` command and specify the VPC ID, type of VPC endpoint \(interface\), service name, subnets to use the endpoint, and security groups to associate with the endpoint network interfaces\. For information about how to set up a security group for your VPC endpoint, see [Configuring Your Security Groups for Amazon EI](#ei-security)\. 

  ```
  aws ec2 create-vpc-endpoint --vpc-id vpc-insert VPC ID --vpc-endpoint-type Interface --service-name com.amazonaws.us-west-2.elastic-inference.runtime --subnet-id subnet-insert subnet --security-group-id sg-insert security group ID
  ```

## Configuring an Instance Role with an Amazon EI Policy<a name="ei-role-policy"></a>

To launch an instance with an Amazon EI accelerator, you must provide an [IAM role](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/iam-roles-for-amazon-ec2.html) that allows actions on Amazon EI accelerators\. 

**To configure an instance role with an Amazon EI policy \(console\)**

1. Open the IAM console at [https://console\.aws\.amazon\.com/iam/](https://console.aws.amazon.com/iam/)\.

1. In the left navigation pane, choose **Policies**, **Create Policy**\. 

1. Choose **JSON** and paste the following policy:

   ```
   	{
   	    "Version": "2012-10-17",
   	    "Statement": [
   	        {
   	            "Effect": "Allow",
   	            "Action": "elastic-inference:Connect",
   	            "Resource": "*"
   	        }
   	    ]
    	}
   ```

1. Choose **Review policy** and enter a name for the policy, such as `ec2-role-trust-policy.json`, and a description\.

1. Choose **Create policy**\.

1. In the left navigation pane, choose **Roles**, **Create role**\.

1. Choose **AWS service**, **EC2**, **Next: Permissions**\.

1. Select the name of the policy that you just created \(`ec2-role-trust-policy.json`\)\. Choose **Next: Review**\.

1. Provide a role name and choose **Create Role**\.

When you create your instance, select the role under **Configure Instance Details** in the launch wizard\.

**To configure an instance role with an Amazon EI policy \(AWS CLI\)**
+ To configure an instance role with an Amazon EI policy, follow the steps in [Creating an IAM Role](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/iam-roles-for-amazon-ec2.html#create-iam-role)\. Add the following policy to your instance:

  ```
  	{
  	    "Version": "2012-10-17",
  	    "Statement": [
  	        {
  	            "Effect": "Allow",
  	            "Action": "elastic-inference:Connect",
  	            "Resource": "*"
  	        }
  	    ]
   	}
  ```

## Launching an Instance with Amazon EI<a name="eia-launch"></a>

You can now configure EC2 instances with accelerators to launch within your subnet\. You can choose any supported Amazon EC2 instance type and Amazon EI accelerator size\. Amazon EI accelerators are available to all current generation instance types\. There are three Amazon EI accelerator sizes to choose from: 
+ `eia1.medium` with 1 GB of accelerator memory
+ `eia1.large` with 2 GB of accelerator memory
+ `eia1.xlarge` with 4 GB of accelerator memory

**To launch an instance with Amazon EI \(console\)**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. Choose **Launch Instance**\. 

1. Under **Choose an Amazon Machine Image**, select an Amazon Linux or Ubuntu AMI\. We recommend one of the Deep Learning AMIs\. 

1. Under **Choose an Instance Type**, select the hardware configuration of your instance\. 

1. Choose **Next: Configure Instance Details**\.

1. Under **Configure Instance Details**, check the configuration settings\. Ensure that you are using the VPC with the security groups for the instance and the Amazon EI accelerator that you set up earlier\. For more information, see [Configuring Your Security Groups for Amazon EI](#ei-security)\.

1. For **IAM role**, select the role that you created in the [Configuring an Instance Role with an Amazon EI Policy](#ei-role-policy) procedure\.

1. Select **Add an Amazon EI accelerator**\.

1. Select the size of the Amazon EI accelerator\. Your options are: `eia1.medium`, `eia1.large`, and `eia1.xlarge`\. 

1. \(Optional\) You can choose to add storage and tags by choosing **Next** at the bottom of the page\. Or, you can let the instance wizard complete the remaining configuration steps for you\. 

1. Review the configuration of your instance and choose **Launch**\. 

1. You are prompted to choose an existing key pair for your instance or to create a new key pair\. For more information, see [Amazon EC2 Key Pairs\.](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-key-pairs.html)\.
**Warning**  
Don’t select the **Proceed without a key pair** option\. If you launch your instance without a key pair, then you can’t connect to it\. 

1. After making your key pair selection, choose **Launch Instances**\. 

1. A confirmation page lets you know that your instance is launching\. To close the confirmation page and return to the console, choose **View Instances**\. 

1. Under **Instances**, you can view the status of the launch\. It takes a short time for an instance to launch\. When you launch an instance, its initial state is `pending`\. After the instance starts, its state changes to `running`\.

1. It can take a few minutes for the instance to be ready so that you can connect to it\. Check that your instance has passed its status checks\. You can view this information in the **Status Checks** column\. 

**To launch an instance with Amazon EI \(AWS CLI\)**

To launch an instance with Amazon EI at the command line, you need your key pair name, subnet ID, security group ID, AMI ID, and the name of the instance profile that you created in the section [Configuring an Instance Role with an Amazon EI Policy](#ei-role-policy)\. For the security group ID, use the one you created for your instance that contains the AWS PrivateLink endpoint\. For more information, see [Configuring Your Security Groups for Amazon EI](#ei-security)\)\. For more information about the AMI ID, see [Finding a Linux AMI](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/find-an-ami.html)\.

1. Use the `run-instances` command to launch your instance and accelerator:

   ```
   aws ec2 run-instances --image-id ami-insert image ID --instance-type for exmample, m5.large --subnet-id subnet-insert your subnet ID --elastic-inference-accelerator Type=for example, eia1.large --key-name enter your key pair name --security-group-ids sg-enter your security group ID --iam-instance-profile Name="enter the name of your accelerator profile"
   ```

1. When the `run-instances` operation succeeds, your output is similar to the following\. The ElasticInferenceAcceleratorArn identifies the Amazon EI accelerator\.

   ```
   "ElasticInferenceAcceleratorAssociations": [
    {
    "ElasticInferenceAcceleratorArn": "arn:aws:elastic-inference:us-west-2:204044812891:elastic-inference-accelerator/eia-3e1de7c2f64a4de8b970c205e838af6b",
    "ElasticInferenceAcceleratorAssociationId": "eia-assoc-031f6f53ddcd5f260",
    "ElasticInferenceAcceleratorAssociationState": "associating",
    "ElasticInferenceAcceleratorAssociationTime": "2018-10-05T17:22:20.000Z"
    }
    ],
   ```

You are now ready to run your models using either TensorFlow or MXNet on the provided AMI\.

## Using TensorFlow Models with Amazon EI<a name="ei-tensorflow"></a>

The Amazon EI enabled version of TensorFlow Serving allows you to use Amazon EI accelerators with no changes to your TensorFlow code\. The Amazon EI TensorFlow Serving is available in the AWS Deep Learning AMI\. You can also download the binary from the [Amazon S3 bucket](https://s3.console.aws.amazon.com/s3/buckets/amazonei-tensorflow/) to build it in to your own Amazon Linux or Ubuntu AMIs, or Docker containers\.

With Amazon EI TensorFlow Serving, the standard TensorFlow Serving interface remains unchanged\. The only difference is that the entry point is a different binary named amazonei\_tensorflow\_model\_server\. 

For more information, see [TensorFlow Serving](https://www.tensorflow.org/serving/)\.

This release of Amazon EI TensorFlow Serving has been tested to perform well and provide cost\-saving benefits with the following deep learning use cases and network architectures \(and similar variants\):


| Use Case | Example Network Topology | 
| --- | --- | 
|  **Image Recognition**  |  Inception, ResNet, MVCNN   | 
|  **Object Detection**  | SSD, RCNN  | 
|  **Neural Machine Translation**  | GNMT  | 

### TensorFlow Serving Example<a name="tensorflow-example"></a>

The following is an example of how to run an inception model with Amazon EI TensorFlow Serving\. Note that if you are downloading Amazon EI TensorFlow Serving to run in your AMI or custom AMI, you must install the following dependencies: OpenSSL for IAM Authentication and the [TensorFlow Serving Python API PIP package](https://www.tensorflow.org/serving/setup#tensorflow_serving_python_api_pip_package)\. These dependencies are pre\-installed on AWS Deep Learning AMIs\.

**Serve and Test Inference with an Inception Model**

1. Download the model\.

   ```
   curl -O https://s3-us-west-2.amazonaws.com/aws-tf-serving-ei-example/inception.zip
   ```

1. Unzip the model\.

   ```
   unzip inception.zip
   ```

1. Download a picture of a husky\.

   ```
   curl -O https://upload.wikimedia.org/wikipedia/commons/b/b5/Siberian_Husky_bi-eyed_Flickr.jpg
   ```

1. Navigate to the folder where `AmazonEI_TensorFlow_Serving` is installed and run the following command to launch the server\.

   ```
   AmazonEI_TensorFlow_Serving_v1.11_v1 --model_name=inception --model_base_path=[directory with the unzipped model]/SERVING_INCEPTION/SERVING_INCEPTION --port=9000
   ```

1. While the server is running in the foreground, launch another terminal session\. Open a new terminal and use your preferred text editor to create a script that has the following content\. Name it `inception_client.py`\. This script will take an image filename as a parameter and get a prediction result from the pre\-trained model\.

   ```
   from __future__ import print_function
   
   import grpc
   import tensorflow as tf
   
   from tensorflow_serving.apis import predict_pb2
   from tensorflow_serving.apis import prediction_service_pb2_grpc
   
   
   tf.app.flags.DEFINE_string('server', 'localhost:9000',
                              'PredictionService host:port')
   tf.app.flags.DEFINE_string('image', '', 'path to image in JPEG format')
   FLAGS = tf.app.flags.FLAGS
   
   
   def main(_):
     channel = grpc.insecure_channel(FLAGS.server)
     stub = prediction_service_pb2_grpc.PredictionServiceStub(channel)
     # Send request
     with open(FLAGS.image, 'rb') as f:
       # See prediction_service.proto for gRPC request/response details.
       data = f.read()
       request = predict_pb2.PredictRequest()
       request.model_spec.name = 'inception'
       request.model_spec.signature_name = 'predict_images'
       request.inputs['images'].CopyFrom(
           tf.contrib.util.make_tensor_proto(data, shape=[1]))
       result = stub.Predict(request, 10.0)  # 10 secs timeout
       print(result)
     print("Inception Client Passed")
   
   if __name__ == '__main__':
     tf.app.run()
   ```

1. Now run the script passing the server location, port, and the husky photo's filename as the paramaters\.

   ```
   python inception_client.py --server=localhost:9000 --image Siberian_Husky_bi-eyed_Flickr.jpg
   ```

For more tutorials and examples, see the [TensorFlow Python API](https://www.tensorflow.org/api_docs/python/)\.

### Additional Requirements and Considerations<a name="tensorflow-additional"></a>

**Model Formats Supported**

Amazon EI supports the TensorFlow saved\_model format via TensorFlow Serving\.

**OpenSSL Requirement**

Amazon EI TensorFlow Serving requires OpenSSL for IAM authentication\. OpenSSL is pre\-installed in the AWS Deep Learning AMI\. If you are building your own AMI or Docker container, you must install OpenSSL \. 
+ Command to install OpenSSL for Ubuntu:

  ```
  sudo apt-get install libssl-dev
  ```
+ Command to install OpenSSL for Amazon Linux:

  ```
  sudo yum install openssl-devel
  ```

**Warmup**

Amazon EI TensorFlow Serving provides a [warmup](https://www.tensorflow.org/serving/saved_model_warmup) feature to preload models and reduce the delay that is typical of the first inference request\. Amazon Elastic Inference TensorFlow Serving only supports warming up the "serving\_default" signature definition\.

**Signature Definitions**

Using multiple [signature definitions](https://www.tensorflow.org/serving/signature_defs) can have a multiplicative effect on the amount of accelerator memory consumed\. If you plan to exercise more than one signature definition for your inference calls, you should test these scenarios as you determine the accelerator type for your application\.

## Using MXNet Models with Amazon EI<a name="ei-mxnet"></a>

The Amazon Elastic Inference enabled version of Apache MXNet lets you use Amazon EI seamlessly, with few changes to your MXNet code\. You can use Amazon EI with the following MXNet API operations:
+ MXNet Python Symbol API
+ MXNet Python Module API

### Install Amazon EI Enabled Apache MXNet<a name="ei-apache"></a>

Amazon EI enabled Apache MXNet is available in the AWS Deep Learning AMI\. A 'pip' package is also available on [Amazon S3](https://s3.console.aws.amazon.com/s3/buckets/amazonei-apachemxnet/) so you can build it in to your own Amazon Linux or Ubuntu AMIs, or Docker containers\. 

### Use Amazon EI with the MXNet Symbol API<a name="install-mxnet"></a>

Pass `mx.eia()` as the context in a call to either the `simple_bind()` or the `bind()` methods\. For information, see [Symbol API](https://mxnet.incubator.apache.org/api/python/symbol/symbol.html)\.

The following example calls the `simple_bind()` method:

```
import mxnet as mx

data = mx.sym.var('data', shape=(1,))
sym = mx.sym.exp(data)

# Pass mx.eia() as context during simple bind operation

executor = sym.simple_bind(ctx=mx.eia(), grad_req='null')
for i in range(10):

  # Forward call is performed on remote accelerator
  executor.forward()
  print('Inference %d, output = %s' % (i, executor.outputs[0]))
```

The following example calls the `bind()` method:

```
import mxnet as mx
a = mx.sym.Variable('a')
b = mx.sym.Variable('b')
c = 2 * a + b
# Even for execution of inference workloads on eia,
# context for input ndarrays to be mx.cpu()
a_data = mx.nd.array([1,2], ctx=mx.cpu())
b_data = mx.nd.array([2,3], ctx=mx.cpu())
# Then in the bind call, use the mx.eia() context
e = c.bind(mx.eia(), {'a': a_data, 'b': b_data})

# Forward call is performed on remote accelerator
e.forward()
```

The following example calls the `bind()` method on a pre\-trained real model \(Resnet\-50\) from the Symbol API:

```
import mxnet as mx
import numpy as np

path='http://data.mxnet.io/models/imagenet/'
[mx.test_utils.download(path+'resnet/50-layers/resnet-50-0000.params'),
mx.test_utils.download(path+'resnet/50-layers/resnet-50-symbol.json'),
mx.test_utils.download(path+'synset.txt')]

ctx = mx.eia()

with open('synset.txt', 'r') as f:
  labels = [l.rstrip() for l in f]

sym, args, aux = mx.model.load_checkpoint('resnet-50', 0)

fname = mx.test_utils.download('https://github.com/dmlc/web-data/blob/master/mxnet/doc/tutorials/python/predict_image/cat.jpg?raw=true')
img = mx.image.imread(fname)
# convert into format (batch, RGB, width, height)
img = mx.image.imresize(img, 224, 224) # resize
img = img.transpose((2, 0, 1)) # Channel first
img = img.expand_dims(axis=0) # batchify
img = img.astype(dtype='float32')
args['data'] = img

softmax = mx.nd.random_normal(shape=(1,))
args['softmax_label'] = softmax

exe = sym.bind(ctx=ctx, args=args, aux_states=aux, grad_req='null')

exe.forward()
prob = exe.outputs[0].asnumpy()
# print the top-5
prob = np.squeeze(prob)
a = np.argsort(prob)[::-1]
for i in a[0:5]:
  print('probability=%f, class=%s' %(prob[i], labels[i]))
```

### Use Amazon EI with the MXNet Module API<a name="ei-mxnet-module-api"></a>

When you create the `Module` object, pass `mx.eia()` as the context\. For more information, see [Module API](https://mxnet.incubator.apache.org/api/python/index.html#module-api)\.

To use the MXNet Module API, you can use the following commands: 

```
# Load saved model
sym, arg_params, aux_params = mx.model.load_checkpoint(model_path, EPOCH_NUM)

# Pass mx.eia() as context while creating Module object
mod = mx.mod.Module(symbol=sym, context=mx.eia())

# Only for_training = False is supported for eia
mod.bind(for_training=False, data_shapes=data_shape)
mod.set_params(arg_params, aux_params)

# Forward call is performed on remote accelerator
mod.forward(data_batch)
```

The following example uses Amazon EI with the Module API on a pre\-trained real model \(Resnet\-152\):

```
import mxnet as mx
import numpy as np
from collections import namedtuple
Batch = namedtuple('Batch', ['data'])

path='http://data.mxnet.io/models/imagenet/'
[mx.test_utils.download(path+'resnet/152-layers/resnet-152-0000.params'),
mx.test_utils.download(path+'resnet/152-layers/resnet-152-symbol.json'),
mx.test_utils.download(path+'synset.txt')]

ctx = mx.eia()

sym, arg_params, aux_params = mx.model.load_checkpoint('resnet-152', 0)
mod = mx.mod.Module(symbol=sym, context=ctx, label_names=None)
mod.bind(for_training=False, data_shapes=[('data', (1,3,224,224))],
     label_shapes=mod._label_shapes)
mod.set_params(arg_params, aux_params, allow_missing=True)
with open('synset.txt', 'r') as f:
  labels = [l.rstrip() for l in f]

fname = mx.test_utils.download('https://github.com/dmlc/web-data/blob/master/mxnet/doc/tutorials/python/predict_image/cat.jpg?raw=true')
img = mx.image.imread(fname)

# convert into format (batch, RGB, width, height)
img = mx.image.imresize(img, 224, 224) # resize
img = img.transpose((2, 0, 1)) # Channel first
img = img.expand_dims(axis=0) # batchify

mod.forward(Batch([img]))
prob = mod.get_outputs()[0].asnumpy()
# print the top-5
prob = np.squeeze(prob)
a = np.argsort(prob)[::-1]
for i in a[0:5]:
  print('probability=%f, class=%s' %(prob[i], labels[i]))
```

This release of Amazon EI Apache MXNet has been tested to perform well and provide cost\-saving benefits with the following deep learning use cases and network architectures \(and similar variants\)\.


| Use Case | Example Network Topology | 
| --- | --- | 
|  **Image Recognition**  |  Inception, ResNet, VGG, ResNext  | 
|  **Object Detection**  | SSD                | 
|  **Text to Speech**  |  WaveNet    | 

### Additional Requirements and Considerations<a name="mxnet-additional"></a>
+ Amazon EI Apache MXNet is built with MKLDNN\. Therefore, all operations are supported when using the `mx.cpu()` context\. The `mx.gpu()` context is not supported, therefore no operations can be performed on a local GPU\.
+ Amazon EI is not currently supported for MXNet Imperative mode or the MXNet Gluon API\.
+ `mx.eia()` does not currently provide the full functionality of an MXNet context\. You cannot allocate memory for NDArray on the Amazon EI accelerator by writing something such as: `x = mx.nd.array([[1, 2], [3, 4]], ctx=mx.eia())`\. This results in an error\. Instead you should use: `x = mx.nd.array([[1, 2], [3, 4]], ctx=mx.cpu())`\. MXNet automatically transfers your data to the accelerator as necessary\.
+ Because Amazon EI only supports inference, the `backward()` method or calls to bind\(\) with `for_training=True`\. Because the default value of `for_training` is `True`, make sure that you set `for_training=False`\. 
+ The Amazon EI Apache MXNet pip package on S3 is tested to work with Amazon Linux 1 and 2, and Ubuntu 16\. If you encounter a `(Caused by SSLError("Can't connect to HTTPS URL because the SSL module is not available.",))` after you install the pip package, resolve this issue by removing the SSL shared object libraries from the installation\. To do this, navigate to where Amazon EI MXNet has been installed and remove `libssl.so.1.0.0` and `libcrypto.so.1.0.0` from the MXNet directory\.

## Using ONNX Models with Amazon EI<a name="ei-ONNX"></a>

You can deploy the Open Neural Network Exchange \(ONNX\) format models for inference using Apache MXNet enhanced by Amazon Elastic Inference\. This allows users who are building models using PyTorch, Chainer, CNTK, and other ONNX\-compatible frameworks to use efficient Amazon Elastic Inference acceleration by exporting models to ONNX\.

For more information about using the AWS Deep Learning AMI to apply ONNX models with MXNet, see [Use Apache MXNet for Inference with an ONNX Model](https://docs.aws.amazon.com/dlami/latest/devguide/tutorial-mxnet-inference-onnx.html)\. Use the following command to activate the environment: `source activate amazonei_mxnet_p36`\. Configure the context as `ctx = mx.eia()`\.

# Using CloudWatch Metrics to Monitor Amazon EI<a name="cloudwatch-metrics-ei"></a>

You can monitor your Amazon EI accelerators using Amazon CloudWatch, which collects metrics about your usage and performance\. These statistics are recorded for a period of two weeks so that you can access historical information and gain a better perspective of how your service is performing\. 

By default, Amazon EI sends metric data to CloudWatch in 5\-minute periods\.

For more information, see the [Amazon CloudWatch User Guide](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/)\.

**Topics**
+ [Amazon EI Metrics and Dimensions](#ei-metrics-dimensions)
+ [Creating CloudWatch Alarms to Monitor Amazon EI](#ei-cw-alarms)

## Amazon EI Metrics and Dimensions<a name="ei-metrics-dimensions"></a>

Metrics are grouped first by the service namespace, and then by the various dimension combinations within each namespace\. You can use the following procedures to view the metrics for Amazon EI\.

**To view metrics using the CloudWatch console**

1. Open the CloudWatch console at [https://console\.aws\.amazon\.com/cloudwatch/](https://console.aws.amazon.com/cloudwatch/)\.

1. If necessary, change the Region\. From the navigation bar, select the region where Amazon EI resides\. For more information, see [Regions and Endpoints](https://docs.aws.amazon.com/general/latest/gr/rande.html)\.

1. In the navigation pane, choose **Metrics**\. 

1. Under **All metrics**, select a metrics category, and then scroll down to view the full list of metrics\. 

**To view metrics \(AWS CLI\)**
+ At a command prompt, enter the following command:

  ```
  aws cloudwatch list-metrics --namespace " AWS/ElasticInference "
  ```

CloudWatch displays the following metrics for Amazon EI\.


| Metric | Description | 
| --- | --- | 
|                     **AcceleratorHealthCheckFailed**                 | Reports whether the Amazon EI accelerator haspassed a status health check in the last minute\. A value ofzero \(0\) indicates that the status check passed\. A value of one \(1\) indicates a status check failure\.Units: Count                 | 
|                     **ConnectivityCheckFailed**                 | Reports whether connectivity to the Amazon EI accelerator is active or has failed in the last minute\. A value of zero \(0\) indicates that the connection is active\. A value of one \(1\) indicates a connectivity failure\.Units: Count                 | 
|                     **AcceleratorMemoryUsage**                 | The memory of the Amazon EI accelerator used in the last minute\.Units: Bytes                 | 

You can filter the Amazon EI data using the following dimensions\. 


| Dimension | Description | 
| --- | --- | 
|                     **ElasticInferenceAcceleratorId**                 | This dimension filters the data by the Amazon EI accelerator\.                 | 
|                     **InstanceId**                 | This dimension filters the data by instance to which the Amazon EI accelerator is attached\.                 | 

## Creating CloudWatch Alarms to Monitor Amazon EI<a name="ei-cw-alarms"></a>

You can create a CloudWatch alarm that sends an Amazon SNS message when the alarm changes state\. An alarm watches a single metric over a time period that you specify\. It sends a notification to an SNS topic based on the value of the metric relative to a given threshold over a number of time periods\.

For example, you can create an alarm that monitors the health of an Amazon EI accelerator\. It sends a notification when the Amazon EI accelerator fails a status health check for three consecutive 5\-minute periods\. 

**To create an alarm for Amazon EI accelerator health status**

1. Open the CloudWatch console at [https://console\.aws\.amazon\.com/cloudwatch/](https://console.aws.amazon.com/cloudwatch/)\.

1. In the navigation pane, choose **Alarms**, **Create Alarm**\.

1. Choose **Amazon EI Metrics**\. 

1. Select the **Amazon EI** and the **AcceleratorHealthCheckFailed** metric and choose **Next**\.

1. Configure the alarm as follows, and then choose **Create Alarm**:
   + Under **Alarm Threshold**, enter a name and description\. For **Whenever**, choose **=>** and enter **1**\. For the consecutive periods, enter **3**\.
   + Under **Actions**, select an existing notification list or choose **New list**\.
   + Under **Alarm Preview**, select a period of 5 minutes\.
   
   # Troubleshooting<a name="ei-troubleshooting"></a>

The following are common errors and troubleshooting steps\.

**Topics**
+ [Issues Launching Accelerators](#ei-issues)
+ [Resolving Connectivity Issues](#ei-resolving-connectivity)
+ [Resolving Unhealthy Status Issues](#ei-unhealthy-status)
+ [Stop and Start the Instance](#ei-stop-start)
+ [Troubleshooting Model Performance](#troubleshoot-model-performance)
+ [Submitting Feedback](#ei-feedback)

## Issues Launching Accelerators<a name="ei-issues"></a>

Ensure that you are launching in a Region where Amazon EI accelerators are available\. For more information, see the [Region Table](https://aws.amazon.com/about-aws/global-infrastructure/regional-product-services/)\.

## Resolving Connectivity Issues<a name="ei-resolving-connectivity"></a>

If you are unable to successfully connect to accelerators, verify that you have completed the following:
+ You have set up a VPC endpoint for Amazon EI for the subnet in which you have launched your instance\.
+ You have configured security groups for the instance and VPC endpoints with outbound rules that allow communications for HTTPS \(Port 443\)\. You have configured the VPC endpoint security group with an inbound rule that allows HTTPS traffic\.
+ You have added an IAM instance role with the "elastic\-inference:Connect" permission to the instance from which you are connecting to the accelerator\.
+ You have checked CloudWatch Logs to verify that your accelerator is healthy\. The EC2 instance details from the Amazon EC2 console contain a link to CloudWatch, which allows you to view the health of its associated accelerator\.

## Resolving Unhealthy Status Issues<a name="ei-unhealthy-status"></a>

If the Amazon EI accelerator is in an unhealthy state, the following are troubleshooting steps that you can use to resolve the issue\. 

## Stop and Start the Instance<a name="ei-stop-start"></a>

If your Amazon EI accelerator is in an unhealthy state, stopping and starting it again is the simplest option\. For more information, see [Stopping and Starting Your Instances](Stop_Start.md#starting-stopping-instances)\.

**Warning**  
When you stop an instance, the data on any instance store volumes is erased\. If you have any data to preserve on instance store volumes, make sure to back it up to persistent storage\.

## Troubleshooting Model Performance<a name="troubleshoot-model-performance"></a>

Amazon EI accelerates operations defined by frameworks like TensorFlow and MXNet\. While Amazon EI accelerates most neural network, math, array manipulation, and control flow operators, there are many operators that Amazon EI does not accelerate\. These include training\-related operators, input/output operators, and some operators in contrib\. 

When a model contains operators that Amazon EI does not accelerate, the framework runs them on the instance\. The frequency and location of these operators within a model graph can have an impact on the model's inference performance with Amazon EI accelerators\. If your model is known to benefit from GPU acceleration and does not perform well on Amazon EI, contact AWS Support or amazon\-ei\-feedback@amazon\.com\. 

## Submitting Feedback<a name="ei-feedback"></a>

Contact AWS Support or send feedback to: amazon\-ei\-feedback@amazon\.com\. 
