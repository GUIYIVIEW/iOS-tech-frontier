Core Image入门教程(swift)
---

> * 原文链接 : [Core Image Tutorial: Getting Started](http://www.raywenderlich.com/76285/beginning-core-image-swift)
* 原文作者 : [Nick Lockwood](http://www.raywenderlich.com/u/nicklockwood)
* 译文出自 :[开发技术前线 www.devtf.cn](http://www.devtf.cn)
* 译者 : [Sam Lau](https://github.com/samlaudev) 
* 校对者: [这里校对者的github用户名](github链接)  
* 状态 :  翻译完

Core Image is a powerful framework that lets you easily apply filters to images. You can get all kinds of effects, such as modifying the vibrance, hue, or exposure. It can use either the CPU or GPU to process the image data and is very fast — fast enough to do real-time processing of video frames!

Core Image filters can also be chained together to apply multiple effects to an image or video frame at once. The multiple filters are combined into a single filter that is applied to the image. 

In this tutorial, you will get hands-on experience playing around with Core Image. You’ll apply a few different filters, and you’ll see how easy it is to apply cool effects to images in real time!

Core Image是一个强大的框架，它能够让你轻松地对图像进行过滤。你能够通过修改图像的饱和度、色调或曝光率来获取各种特效。你也可以使用CPU或GPU更快地来处理图像数据，快到能够实时处理视频帧数据(video frame)。

Core Image过滤器能够以链式的方式结合将多个特效应用到一个图像或视频帧数据。多个过滤器能够组合成单个过滤器应用在一个图像。通过每次与每个过滤器比较，这使它非常有效地处理图像。

在这个教程中，你将会亲手实践如何使用Core Image.通过使用几个不同的过滤器，你会看到实时使用各种炫酷的特效是多么容易。

#Getting Started
#基础
Before you get started, let’s discuss some of the most important classes in the Core Image framework:

* **CIContext**. All of the processing of a core image is done in a CIContext. This is somewhat similar to a Core Graphics or OpenGL context.
* **CIImage**. This class hold the image data. It can be created from a UIImage, from an image file, or from pixel data.
* **CIFilter**. The CIFilter class has a dictionary that defines the attributes of the particular filter that it represents. Examples of filters are vibrance, color inversion, cropping, and many more.

You’ll be using each of these classes in this project.

在你开始之前，让我们先讨论在Core Image框架中几个很重要的类：

* **CIContext**. 所有处理core image的工作都在CIContext完成。这个有几分与Core Graphics或OpenGL context相似
* **CIImage**. 这个类保存图像数据，它能够从UIImage，图像文件或像素数据等方式来创建。
* **CIFilter**. CIFilter类有一个字典(dictionary)来定义特定属性的过滤器，比如常用过滤器有饱和度，颜色翻转，裁剪等属性，还有很多没列出来。

###CoreImageFun
Open up Xcode and create a new project with the iOS \ Application \ Single View Application template. Enter oreImageFun for the Product Name, select iPhone for the Devices option, and make sure that Language is set to Swift.

Download the resources for this tutorial, and add the included image.png to your project.

Next, open Main.storyboard, and drag an image view in as a subview of the existing view. In the Attributes Inspector, set the image view’s content mode to Aspect Fit, so it won’t distort images.

Next, ensure the Document Outline (the hierarchy left of the canvas in Interface Builder) is visible – you can enable it from the menu at Editor \ Show Document Outline.

Control-drag from the image view to its superview three times to add three constraints:

1. Add a constraint Top Space to Layout Guide, using the Size Inspector to set the constraint’s constant to zero if necessary.
2. Add a constraint to Center Horizontally in Container (also setting its constant to zero).
3. Add an Equal Width constraint.

Finally, to constrain the image view’s height, control drag from the image view to itself, and add an Aspect Ratio constraint, using the Size Inspector to set its multiplier 8:5 for the ratio of width to height and a constant factor of zero. Finally, navigate to Editor \ Resolve Auto Layout Issues \ All Views in View Controller \ Update Frames, so Interface Builder updates the layout based on these new constraints.

Next, open the Assistant Editor and make sure it’s displaying ViewController.swift. Control-drag from the UIImageView to just after the opening brace of the ViewController class. Name the outlet imageView, and click Connect.

Build and run the project just to make sure everything is good so far – you should just see an empty screen. The initial setup is complete – now onto Core Image!

###CoreImageFun
打开Xcode并使用**iOS \ Application \ Single View Application**模板来创建一个新工程。输入**CoreImageFun**作为Product Name，选择iPhone为Devices option和确定使用**swift**作为编程语言。

下载[教程的资源](http://cdn5.raywenderlich.com/downloads/CIResources.zip)，并添加图片**image.png**到工程

下一步，打开**Main.storyboard**你文件，拖拽一个image view到已存在的view，image view作为view的子视图。在Attributes Inspector中，设置image view的content mode属性为Aspect Fit，这样它就不会扭曲图片。

下一步，确定Document Outline(在Interface Builder里canvas左边)是可见的 - 你可以从menu中**Editor \ Show Document Outline**来启用。

按着control键从image view拖动到它的superview三次来添加三个约束(constraint)：

1. 添加一个Top Space to Layout Guide的约束，如果有必要的话，使用Size Inspector来设置约束的constant为0 
2. 添加一个Center Horizontally in Container的约束(同样也需要设置constant为0)
3. 添加一个Equal Width约束

最后，为了约束image view的高度，按着control键从image view拖动到本身，然后添加一个Aspect Ratio约束，使用Size Inspector来设置它的multiplier的值为8:5对应宽高比和constant factor为0。最后，选择**Editor \ Resolve Auto Layout Issues \ All Views in View Controller \ Update Frames**，那么Interface Builder会根据这些约束来更新布局

下一步，打开Assistant Editor，然后确保它显示**ViewController.swift**。按着control键从UIImageView拖动到刚刚打开的**ViewController**类的大括号。命名outlet为**imageView**，然后点击connect。

编译和运行这个项目来确保目前进展顺利 - 你应该看到一个空屏。初始化设置完成，现在进入Core Image的世界。


#Basic Image Filtering
You’re going to get started by simply running your image through a CIFilter and displaying it on the screen. Every time you want to apply a CIFilter to an image you need to do four things:

1. Create a CIImage object. CIImage has several initialization methods, including: CIImage(contentsOfURL:), CIImage(data:), CIImage(CGImage:), CIImage(bitmapData:bytesPerRow:size:format:colorSpace:), and several others. You’ll most likely be working with CIImage(contentsOfURL:) most of the time.
2. Create a CIContext. A CIContext can be CPU or GPU based. A CIContext is relatively expensive to initialize so you reuse it rather than create it over and over. You will always need one when outputting the CIImage object.
3. Create a CIFilter. When you create the filter, you configure a number of properties on it that depend on the filter you’re using.
4. Get the filter output. The filter gives you an output image as a CIImage – you can convert this to a UIImage using the CIContext, as you’ll see below.

你将会通过使用**CIFilter**应用到图像和显示到屏幕来入门。每次你想将CIFilter应用到一个图像，都需要做四件事：

1. **创建一个CIImage对象**。CIImage有几个初始化方法，其中包括：CIImage(contentsOfURL:), CIImage(data:), CIImage(CGImage:), CIImage(bitmapData:bytesPerRow:size:format:colorSpace:)等，大多数你都会使用CIImage(contentsOfURL:)方法。
2. **创建一个CIContext对象**。一个CIContext是基于CPU或CPU，在初始化它时比较耗资源，所以需要复用它而不是多次创建。当你输出CIImage对象时，你会经常需要CIContext对象。
3. **创建CIFilter对象**。当你创建一个filter时，你需要配置多个属性来决定你使用的filter。
4. **获取filter的输出**。filter会输出一个CIImage类型的image给你 - 你可以使用CIContext将它转换为UIImage类型image。

Let’s see how this works. Add the following code to ViewController.swift inside viewDidLoad():

让我们看看它是如何工作。添加以下代码到**ViewController.swift**的viewDidLoad():方法

```
// 1
let fileURL = NSBundle.mainBundle().URLForResource("image", withExtension: "png")
 
// 2
let beginImage = CIImage(contentsOfURL: fileURL)
 
// 3
let filter = CIFilter(name: "CISepiaTone")
filter.setValue(beginImage, forKey: kCIInputImageKey)
filter.setValue(0.5, forKey: kCIInputIntensityKey)
 
// 4
let newImage = UIImage(CIImage: filter.outputImage)
self.imageView.image = newImage
```

Let’s go over this section by section:

1. This line creates an NSURL object that holds the path to your image file.
2. Next you create your CIImage with the CIImage(contentsOfURL:) constructor. 
3. Next you’ll create your CIFilter object. The CIFilter constructor takes the name of the filter, and a dictionary that specifies the keys and values for that filter. Each filter will have its own unique keys and set of valid values. The CISepiaTone filter takes only two values, the KCIInputImageKey (a CIImage) and the kCIInputIntensityKey, a float value between 0 and 1. Here you give that value 0.5. Most of the filters have default values that will be used if no values are supplied. One exception is the CIImage, this must be provided as there is no default.
4. Getting a CIImage back out of a filter is as easy as using the outputImage property. Once you have an output CIImage, you will need to convert it into a UIImage. The UIImage(CIImage:) constructor creates a UIImage from a CIImage. Once you’ve converted it to a UIImage, you just display it in the image view you added earlier.

Build and run the project, and you’ll see your image filtered by the sepia tone filter.

让我们逐段分析以上代码：

1. 这行代码创建一个NSURL对象，它保存image文件的路径
2. 通过CIImage(contentsOfURL:)构造器来创建CIImage对象
3. 创建CIFilter对象。CIFilter构造器将name作为参数，然后用dictionary来指定filter的键值对。每个filter都拥有唯一的键和多个有效值。**CISepiaTone** filter只接收两个值**kCIInputImageKey**(CIImage)和**kCIInputIntensityKey**(在0到1之间的一个浮点数)。这里你设置它的值为0.5。如果没有值提供的话，大多数filters都有默认值。但CIImage没有默认值，它必须要提供。
4. 通过使用filter的**outputImage**属性能够轻松地获取CIImage。一旦你有个输出的CIImage，你需要将它转换为UIImage。使用**UIImage(CIImage:)**构造器来创建UIImage。如果你已经将CIImage转换为UIImage，你就能将image显示到image view

<div align="center">

<img src="http://cdn5.raywenderlich.com/wp-content/uploads/2014/07/CI-Sepia-Crop.jpg"/>
</div>

#Putting It Into Context
#把它放进Context
Before you move forward, there’s an optimization that you should know about.

在你向前看之前，有个优化建议你应该需要知道。

I mentioned earlier that you need a CIContext in order to apply a CIFilter, yet there’s no mention of this object in the above example. It turns out that the the UIImage(CIImage:) constructor does all the work for you. It creates a CIContext and uses it to perform the work of filtering the image. This makes using the Core Image API very easy.

之前我曾提及过，你需要有个**CIContext**来使用CIFilter，但以上例子都没有提及过这个对象。结果变成了由**UIImage(CIImage:)** 构造器来为你处理所有的工作。UIImage它创建一个**CIContext**，然后用它来执行所有过滤图像的操作，这让你更加容易地使用Core Image的API。

There is one major drawback – it creates a new CIContext every time it’s used. CIContext instances are meant to be reusable to increase performance. If you want to use a slider to update the filter value, as you’ll be doing in this tutorial, creating a new CIContext each time you change the filter would be way too slow.

但这种方式有一个主要缺点就是每次使用时都创建一个新的**CIContext**对象。**CIContext**对象是为了复用来提高性能。如果你想用一个slider来更新filter的值，是以每次创建一个新的CIContext来实现更新filter的方式，那么运行速度将会很慢。

Let’s do this properly. Delete step 4 from the code you added to viewDidLoad(), and replace it with the following:

让我们以恰当的方式来实现。在**viewDidLoad()**里删除步骤4的代码，然后用一下代码来代替：

```
// 1
let context = CIContext(options:nil)
 
// 2
let cgimg = context.createCGImage(filter.outputImage, fromRect: filter.outputImage.extent())
 
// 3
let newImage = UIImage(CGImage: cgimg)
self.imageView.image = newImage
```
Again, let’s go over this section by section.

1. Here you set up the CIContext object and use it to draw a CGImage. The CIContext(options:) constructor takes an NSDictionary that specifies options such as the color format, or whether the context should run on the CPU or GPU. For this app, the default values are fine and so you pass in nil for that argument.
2. Calling createCGImage(outputImage:fromRect:) on the context with the supplied CIImage will return a new CGImage instance.
3. Next, you use the UIImage(CGImage:) constructor to create a UIImage from the newly created CGImage instead of directly from the CIImage as before. Note that there is no need to explicitly release the CGImage after we are finished with it, as there would have been in Objective-C. In Swift, ARC can automatically release Core Foundation objects.

我们再一次逐段分析代码：

1. 你设置CIContext对象，然后用它来画一个CGImage。**CIContext(options:)**构造器以NSDictionary为参数，它指定一些选项，比如：颜色格式，或者context是否运行在CPU或GPU。对于这个app来说，默认值是可以的，所以你可以传递nil这个值进去。
2. context调用**createCGImage(outputImage:fromRect:)**方法在给定CIImage参数并返回一个新的CGImage示例。
3. 你根据刚刚获取的CIImage，使用**UIImage(CGImage:)**构造器来创建一个UIImage对象。注意，当我们使用CIImage对象之后，没有必要显式地释放它，虽然在Objective-C需要这样做。但在Swift中，ARC会自动释放Core Foundation的对象。


Build and run, and make sure it works just as before.

编译和运行，确保项目进展顺利

In this example, handling the CIContext creation yourself doesn’t make much difference. But in the next section, you’ll see why this is important for performance, as you implement the ability to modify the filter dynamically!

在这个例子中，自己创建CIContext与不创建没什么不同。但下个部分中，你会看到当你动态地修改filter时，为什么CIContext对性能影响很大。

#Changing Filter Values
#修改Filter值
This is great, but it’s just the beginning of what you can do with Core Image filters. Lets add a slider and set it up so you can adjust the filter settings in real time.

目前还不错，但这只是使用Core Image filters的入门方式。让我们添加一个slider，然后设置它以便你能够实时调整filter的设置

Open Main.storyboard, and drag in a slider, and drop it in below the image view, centered horizontally. With the view selected, navigate to Editor \ Resolve Auto Layout Issues \ Selected Views \ Reset to Suggested Constraints, increasing the width constraint if desired.

打开**Main.storyboard**，选取一个slider，将它拖放到image view的上面，并水平对齐。选中view，然后点击**Editor \ Resolve Auto Layout Issues \ Selected Views \ Reset to Suggested Constraints**，添加需要的宽约束。

Make sure the Assistant Editor is visible and displaying ViewController.swift, then control-drag from the slider down below the previously added @IBOutlet, set the name to amountSlider, and click Connect.

确保Assistant Editor可见和显示**ViewController.swift**，然后按着control键从slider拖动到之前添加的@IBOutlet下面，设置name为**amountSlider**，然后点击**Connect**。

While you’re at it let’s connect the slider to an action method as well. Again control-drag from the slider, this time to just above the closing } of the ViewController class. Set the Connection to Action, the name to amountSliderValueChanged, make sure that the Event is set to Value Changed, and click Connect.

当你还选中slider时，也让我们连接slider到一个action method。再一次按着control键从slider拖动到ViewController类的}上面。设置Connection为**Action**，name为**amountSliderValueChanged**，确保Event设置为**Value Changed**，然后点击**Connect**。

Every time the slider changes, you need to redo the image filter with a different value. However, you don’t want to redo the whole process, that would be very inefficient and would take too long. You’ll need to change a few things in your class so that you hold on to some of the objects you create in your viewDidLoad method.

每次slider改变时，你需要根据不同的值来重新创建image filter。然而，你不想重复整个耗时且没效率的过程。你需要在你的类改变几样东西，那么你就要在viewDidLoad方法中创建一些对象并保存。

The biggest thing you want to do is reuse the CIContext whenever you need to use it. If you recreate it each time, your program will run very slowly. The other things you can hold onto are the CIFilter and the CIImage that holds your original image. You’ll need a new CIImage for every output, but the image you start with will stay constant.

最重要的一件事就是当你需要使用CIContext时，你只需复用它。如果你每次都重新创建它，你的程序将会变得很慢。另一件事就是保存CIFilter和CIImage，CIImage主要保存原始的图像。每次输出都会产生新的CIImage，但你刚开始使用的图像都会保持不变。

You need to add some instance variables to accomplish this task. Add the following three properties to your ViewController class:

你需要添加几个实例变量来完成这个任务。添加以下三个属性到你的ViewController类：

```
var context: CIContext!
var filter: CIFilter!
var beginImage: CIImage!

```

Note that you have declared these values as implicitly-unwrapped optionals using the ! syntax, because you aren’t going to initialize them until viewDidLoad. You could have used ? instead, but you know that the way the code is designed will prevent the optionals from being nil by the time you use them. The implicitly-unwrapped syntax makes it much easier to read, without all the exclamation marks everywhere.

请注意，你已经用!语法来声明那些值为implicitly-unwrapped optionals，因为直到**viewDidLoad**才初始化它们。你也可以用？，但采用那种方式是为了当你使用那几个实例变量时，防止optionals为nil。implicitly-unwrapped语法由于不用到处使用!标识来访问变量，它让代码更加易读。

Change the code in viewDidLoad so it uses these properties instead of declaring new local variables, as follows:

在**viewDidLoad**改变代码，那么它使用这些属性而不是使用新的局部变量，代码如下：

```
beginImage = CIImage(contentsOfURL: fileURL)
 
filter = CIFilter(name: "CISepiaTone")
filter.setValue(beginImage, forKey: kCIInputImageKey)
filter.setValue(0.5, forKey: kCIInputIntensityKey)
 
let outputImage = filter.outputImage
 
context = CIContext(options:nil)
let cgimg = context.createCGImage(outputImage, fromRect: outputImage.extent())

```

Now you’ll implement the changeValue method. What you’ll be doing in this method is altering the value of the **inputIntensity** key in your CIFilter dictionary.

现在你将会实现changeValue方法。你在这个方法需要做的是修改CIFilter dictionary的inputIntentsity键对应的值。

Once you’ve altered this value you’ll need to repeat a few steps:

* Get the output CIImage from the CIFilter.
* Convert the CIImage to a CGImage.
* Convert the CGImage to a UIImage, and display it in the image view.

一旦你修改这个值，你需要重复这几个步骤：

* 从CIFilter获取输出的CIImage
* 将CIImage转换为CGImage
* 将CGImage转换为UIImage，然后将它显示在image view

Replace amountSliderValueChanged(sender:) with the following:

用以下代码代替amountSliderValueChanged(sender:)方法：

```
@IBAction func amountSliderValueChanged(sender: UISlider) {
 
    let sliderValue = sender.value
 
    filter.setValue(sliderValue, forKey: kCIInputIntensityKey)
    let outputImage = filter.outputImage
 
    let cgimg = context.createCGImage(outputImage, fromRect: outputImage.extent())
 
    let newImage = UIImage(CGImage: cgimg)
    self.imageView.image = newImage
}

```

You’ll notice that you’ve changed the argument type from AnyObject to UISlider in the method definition. You know you’ll only be using this method to retrieve values from your UISlider, so you can go ahead and make this change. If you’d left it as AnyObject, you’d need to cast it to a UISlider or the next line would throw an error.

你会注意到，你已经将方法定义中参数类型从**AnyObject**转换为**UISlider**。你只用这个方法来从**UISlider**获取值，以便你改变值。如果你不管它，默认是**AnyObject**，你需要将它转换为**UISlider**，否则下一行代码就会抛出错误。

You retrieve the value from the slider (which returns a Float). Your slider is set to the default values – min 0, max 1, default 0.5. How convenient, these happen to be the right values for this CIFilter!

你可以从slider获取**浮点数**的值。你的slider默认设置为0.5，最小值为0，最大值为1。通过slider设置CIFilter是多么方便。

The CIFilter has methods that will allow you to set the values for the different keys in its dictionary. Here, you’re just setting the inputIntensity to whatever you get from your slider. Swift automatically converts the primitive CFloat value into an NSNumber object suitable for use with setValue(value:forKey:).

CIFilter有多个方法允许你在dictionary根据不同键来设置多个值。而这里，你刚设置**inputIntensity**键对应的值，而这个值是从slider获取。Swift自动将CGFloat值转换为NSNumber对象，来符合**setValue(value:forKey:)**方法的使用。

The rest of the code should look familiar, as it follows the same logic as viewDidLoad. You’re going to be using this code over and over again. From now on, you’ll use amountSliderValueChanged(sender:) to render the output of a CIFilter to your UIImageView.

剩下的代码看起来很熟悉，因为它与**viewDidLoad**方法的逻辑一样。你将会多次使用这段代码。从现在起，你会用**amountSliderValueChanged(sender:)**方法来将CIFilter输出的图像渲染到ImageView。

Build and run, and you should have a functioning live slider that will alter the sepia value for your image in real time!

编译和运行，你可以修改slider值来实时地改变图像。

#Getting Photos from the Photo Album
#从相册获取图片
Now that you can change the values of the filter on the fly, things are starting to get real interesting! But what if you don’t care for this image of flowers? Next you’ll set up a UIImagePickerController so you can get pictures from out of the photo album and into your app so you can play with them.

现在你可以改变filter的值，事情也开始变得有趣。但是，如果你不喜欢这张花朵的图片。你可以设置**UIImagePickerController**来从相册选取图片放进你的app来使用。

You need to create a button that will bring up the photo album view, so open up Main.storyboard, drag in a button to the bottom right of the scene, and label it “Photo Album”. As before, use Auto Layout to Reset to Suggested Constraints. The button should be underneath the slider on the right side.

你需要创建一个button来跳转到相册视图，所以打开**Main.storyboard**，拖动一个button到scene的右底部，并改变按钮文字为"Photo Album"。像之前一样，使用Auto Layout来Reset to Suggested Constraints。button应该在slider的右下边。

Make sure the Assistant Editor is visible and displaying ViewController.swift, then control-drag from the button down to just above the closing } for the ViewController class. Set the Connection to Action, the name to loadPhoto, make sure that the Event is set to Touch Up Inside, and click Connect.

确保Assistant Editor是可见和显示**ViewController.swift**，然后按着control键从button拖动到}的上面。设置Connection为**Action**，name为**loadPhoto**，确保Event设置为**Touch Up Inside**，最后点击**Connect**。

Implement the loadPhoto method as follows:

loadPhoto方法实现如下：

```
@IBAction func loadPhoto(sender : AnyObject) {
  let pickerC = UIImagePickerController()
  pickerC.delegate = self
  self.presentViewController(pickerC, animated: true, completion: nil)
}

```

The first line of code instantiates a new UIImagePickerController. You then set the delegate of the image picker to self (the ViewController).

第一行代码主要是创建一个**UIImagePickerController**对象。然后设置image picker的delegate为self(ViewController)

You get a warning here. You need to declare that ViewController conforms to the UIImagePickerControllerDelegate and UINavigationControllerDelegate protocols.

你会这里得到一个警告。你需要声明**ViewController**遵循**UIImagePickerControllerDelegate**和**UINavigationControllerDelegate**协议。

Still in ViewController.swift, change the class definition at the top of the file as follows:

仍在ViewController.swift文件，在文件顶部改变类的定义，代码如下：

```
class ViewController: UIViewController, UINavigationControllerDelegate, UIImagePickerControllerDelegate {
```
Now implement the following method:

而方法实现如下：

```
func imagePickerController(picker: UIImagePickerController!, didFinishPickingMediaWithInfo info: NSDictionary!) {
  self.dismissViewControllerAnimated(true, completion: nil);
  println(info);
}

```

This UIImagePickerControllerDelegate method isn’t completed yet – it’s just a placeholder to log some information about the chosen image. Note that whenever you implement any of the UIImagePickerControllerDelegate methods, you have to dismiss the UIImagePickerController explicitly in your implementation. If you don’t, then you’ll just stare at the image picker forever!

**UIImagePickerControllerDelegate**方法还没完成 - 它只是一个占位符来打印被选择图片的信息。注意，不管你怎样实现UIImagePickerControllerDelegate这个方法，你必须在实现中显式地dimiss UIImagePickerController。如果你不这样做的话，你就会永远地盯着image picker。


Build and run the app, and tap the button. It will bring up the image picker with the photos in your photo album.

编译和运行这个app，点击button。它会跳转到相册任你选择图片。

> If you are running this in the simulator, you probably won’t have any photos. On the simulator (or on a device without a camera), you can use Safari to save images to your photo album. Open Safari, find an image, tap and hold, and you’ll get an option to save that image. Next time you run your app, it will appear in your photo library.

> 如果你在模拟器运行app，你可以不会有图片。在模拟器(或在没有摄像头的设备)，你可以使用Safari来保存图片到相册。打开Safari，查找一张图片，点着并长按，你就会有一个选项来保存图片。下一次你运行你app，它将会出现在你图片库。

Here’s what you should see in the console after you’ve selected an image (something like this):

在控制台中，你选择完一张图片之后，就看到类似以下的打印信息：

```
{
UIImagePickerControllerMediaType = "public.image";
UIImagePickerControllerOriginalImage = " size {1165, 770} orientation 0 scale 1.000000";
UIImagePickerControllerReferenceURL = "assets-library://asset/asset.PNG?id=DCFE1435-2C01-4820-9182-40A69B48EA67&ext=PNG";
}

```

Note that it has an entry in the dictionary for the “original image” selected by the user. This is what you want to pull out and filter!

注意，它有一个dictionary入口，对应就是用户选择的“原始图片”。这个就是你想拽取和过滤的东西。

Now that you’ve got a way to select an image, how do you use that as your beginImage?

现在你已经有方法选取图片了，你怎样使用它作为你的**beginImage**呢？

Simple, just update the delegate method to look like this:

很简答，只需将delegate方法修改成以下代码：

```
func imagePickerController(picker: UIImagePickerController!, didFinishPickingMediaWithInfo info: NSDictionary!) {
  self.dismissViewControllerAnimated(true, completion: nil);
 
  let gotImage = info[UIImagePickerControllerOriginalImage] as UIImage
 
  beginImage = CIImage(image:gotImage)
  filter.setValue(beginImage, forKey: kCIInputImageKey)
  self.amountSliderValueChanged(amountSlider)
}

```
You need to create a new CIImage from your selected photo. You can get the UIImage representation of the photo by finding it in the dictionary of values, under the UIImagePickerControllerOriginalImage key constant. Note that it’s better to use a constant for this, rather than a hardcoded string, because Apple could change the name of the key in the future.

你需要从你选择的图片来创建一个新的**CIImage**。你可以通过在dictionary的**UIImagePickerControllerOriginalImage**键获取值，从而获取**UIImage**的表示。注意，最好就是用UIImagePickerControllerOriginalImage这个常量，而不是硬编码的字符串，因为Apple可能在将来会改变这个键的名字。

You need to convert the image into a CIImage object with the CIImage(image:) constructor. You then set the key in the filter dictionary so that the input image is the new CIImage you just created.

你需要通过**CIImage(image:)**构造器将image转换为**CIImage**对象。然后在filter dictionary设置键，那么就可以创建新的CIImage。

The last line may seem odd. Remember how I pointed out that the code in changeValue ran the filter with the latest value and updated the image view with the result?

最后一行代码看起来有点奇怪。还记得我说过，怎样运行**changeValue**方法，最新的值被设置在filter，然后更新image view。

Well you need to do that again, so you can just call changeValue. Even though the slider value hasn’t changed, you can still use that method’s code to get the job done. You could break that code into its own method (and if you were going to be working with more complexity, you would, to avoid confusion), but in this case your purpose is served by re-using the amountSliderValueChanged method. Pass in the amountSlider as the sender so that it has the correct value to use.

好，你需要再做一次，所以你只是调用**changeValue**方法。即使slider的值还没改变，你仍然可以使用那个方法的代码来完成工作。你可以分解那段代码到自己的方法(如果你想做得更加复杂，但避免混淆)，但这种情况下，你的目的是为了复用amountSliderValueChanged方法。传递**amountSlider**作为sender以致它有正确的值使用。

Build and run, and now you’ll be able to update any image from your photo album!

编译和运行，你能从相册中更新任何图片。

What if you create the perfect sepia image, how do you hold on to it? You could take a screenshot, but the proper way is to save the filtered photo back into the photo album.

如果你创建完美的深褐色图片，你怎样才能保存它。你可以截图，但最恰当的方式就是保存已过滤的图片到相册。

#Saving to Photo Album
#保存到相册
To save to the photo album, you need to use the AssetsLibrary framework. Add the following import statement to the top of ViewController.swift:

为了保存到相册，你需要使用**AssetsLibrary** framework。在**ViewController.swift**文件顶部添加以下导入语句：

```
import AssetsLibrary
```
One thing you should know is that when you save a photo to the album, it may take a few seconds, and that process could continue even after you close the app.

有一件你需要知道的事就是当你保存图片到相册，它可能花费几秒和在你关闭app之后仍在继续处理。

This could be a problem, as the GPU stops whatever it’s doing when you switch from one app to another. If the photo hasn’t finished saving, it won’t be there when you go looking for it later!

由于当你切换到另一个app的时候，GPU就停止处理，这将变成一个问题。如果图片还没完成保存，当你迟点找到它的时候，它将不在那里。

The solution to this is to use a CPU-based CIContext for rendering. The default behavior is to use the GPU because it’s much faster, and you don’t want to slow down the filtering performance for the sake of adding save functionality. Instead, you will create a second CIContext to use for saving the image. Note that the software renderer won’t work properly in the simulator.

有一个解决方案就是使用基于CPU的**CIContext**来渲染。默认选择就是使用GPU，因为它处理速度更快，但你不想为了添加保存功能而降低过滤性能。而现在，你会创建另一个**CIContext**来保存image。注意，软件渲染将不会在模拟器正常工作。

Add a new button to your app that will let you save the photo you are currently modifying with all the changes you’ve made. Open Main.storyboard, add a new button labeled “Save to Album”. Place the button under the slider on the left side, and add the suggested constraints.

添加一个新button到你的app，这个button会让你保存修改后的图片。打开**Main.storyboard**，添加一个新的button，修改button标题为“Save to Album”。将button放在slider的左边，然后添加相应的约束。

Then connect it to a new savePhoto(sender:) method, like you did last time, and implement the method as follows:

然后像上一次一样，连接button到一个新方法**savePhoto(sender:)**，方法实现如下：

```
@IBAction func savePhoto(sender: AnyObject) {
    // 1
    let imageToSave = filter.outputImage
 
    // 2
    let softwareContext = CIContext(options:[kCIContextUseSoftwareRenderer: true])
 
    // 3
    let cgimg = softwareContext.createCGImage(imageToSave, fromRect:imageToSave.extent())
 
    // 4
    let library = ALAssetsLibrary()
    library.writeImageToSavedPhotosAlbum(cgimg,
      metadata:imageToSave.properties(),
      completionBlock:nil)
}

```
In this code block you:

1. Get the CIImage output from the filter.
2. Create a new, software based CIContext that uses the CPU renderer.
3. Generate the CGImage.
4. Save the CGImage to the photo library.

在这段代码块中：

1. 从filter获取CIImage的输出。
2. 创建一个新的，基于软件的，使用CPU渲染器的CIContext对象。
3. 生成CGImage。
4. 保存CIImage到photo library。

Build and run the app (remember to run on an actual device, since you’re using software rendering), and now you can save that “perfect image” to your photo library so it’s preserved forever!

编译和运行app(记住要运行在实际设备，因为你已经使用软件渲染)，现在你可以永久地保存完美的图像到你的photo library。

#What About Image Metadata?
#Image元数据是什么？
Let’s talk about image metadata for a moment. Image files taken on mobile phones have a variety of data associated with them, such as GPS coordinates, image format, and orientation.

让我们讨论一下关于image元数据。用手机拍照出来的图片有很多关联它的数据，例如GPS坐标，图片格式，和方向。

Orientation in particular is something that you’ll need to preserve. The process of loading a UIImage into a CIImage, rendering to a CGImage, and converting back to a UIImage strips the metadata from the image. In order to preserve orientation, you’ll need to record it and then pass it back into the UIImage constructor.

方向是一样特殊的东西你需要保存。加载UIImage变成CIImage，渲染成CGImage，然后转换回一个UIImage这个过程是为了从image抓取元数据。为了保存方向，你需要保存它然后传递它作为UIImage构造器参数。

Start by adding a new property inside the ViewController class definition:

添加一个新属性到**ViewController**类定义：

```
var orientation: UIImageOrientation = .Up
```

Next, add the following line to imagePickerController(picker:didFinishPickingMediaWithInfo:) just before setting beginImage:

下一步，添加下面代码到**imagePickerController(picker:didFinishPickingMediaWithInfo:)**方法里，设置**beginImage**代码行的上面：

```
orientation = gotImage.imageOrientation

```

This will save the original image orientation to the property.

这样就会保存原始image方向的属性。

Finally, alter the line in amountSliderValueChanged that creates the UIImage that you set to the imageView object:

最后，修改在**amountSliderValueChanged**方法里，那段创建UIImage来设置imageView对象的代码：

```
let newImage = UIImage(CGImage: cgimg, scale:1, orientation:orientation)

```
Now, if you take a picture taken in something other than the default orientation, it will be preserved.

现在，如果你拍一张照片而不是默认方向，它将会默认保存。

#What Other Filters are Available?
#还有哪些Filters是可用的？

The CIFilter API has more than 160 filters on Mac OS, 126 of which are available on iOS 8. As of iOS 8, it is now possible to create you own custom filters as well.

CIFilter API在Mac OS有超过160个，在iOS 8有126个关于filters的API。

In order to find out what filters are available on a given device, you can use the CIFilter method filterNamesInCategory(kCICategoryBuiltIn). This method will return an array of filter names.

为了查找有哪些filters在给定的设备是可用的，你可以使用CIFilter的**filterNamesInCategory(kCICategoryBuiltIn)**方法。这个方法会返回一个关于filter名字的数组。

In addition, each filter has an attributes() method that will return a dictionary containing information about that filter. This information includes the filter’s name and category, the kinds of inputs the filter takes, and the default and acceptable values for those inputs.

除此之外，每个filter有一个**attributes()**方法返回一个dictionary，dictionary包含关于filter的信息。这些信息包括filter的名字，分类，filter接受哪些输入，默认输入，和这些输入接受哪些值。

Let’s put together a method for your class that logs the information for all the currently available filters. Add this method inside the ViewController class definition:

让我们将所有东西放在一个类的方法，这个方法记录当前所有可用的filters信息。添加这个方法到ViewController类的定义：

```
func logAllFilters() {
  let properties = CIFilter.filterNamesInCategory(kCICategoryBuiltIn)
  println(properties)
 
  for filterName: AnyObject in properties {
    let fltr = CIFilter(name:filterName as String)
    println(fltr.attributes())
  }
}

```
This method simply gets the array of filters from filterNamesInCategory(). It prints the list of names first. Then, for each name in the list, it instantiates the filter and logs its attributes dictionary.

这个方法仅仅通过**filterNamesInCategory()**方法来获取filters的数组。首先，它打印名字的列表。然后，遍历列表获取每个名字，根据名字实例化filter和打印它的属性dictionary。

Call this method at the end of viewDidLoad():

在**viewDidLoad()**方法最后调用这个方法：

```
self.logAllFilters()
```

You will see the many filters listed in the console like the following:

你会看到很多filters在控制台被列出：

```
[CIAttributeFilterDisplayName: Color Monochrome, inputColor: {
    CIAttributeClass = CIColor;
    CIAttributeDefault = "(0.6 0.45 0.3 1)";
    CIAttributeType = CIAttributeTypeColor;
}, inputImage: {
    CIAttributeClass = CIImage;
    CIAttributeType = CIAttributeTypeImage;
}, CIAttributeFilterCategories: (
    CICategoryColorEffect,
    CICategoryVideo,
    CICategoryInterlaced,
    CICategoryNonSquarePixels,
    CICategoryStillImage,
    CICategoryBuiltIn
), inputIntensity: {
    CIAttributeClass = NSNumber;
    CIAttributeDefault = 1;
    CIAttributeIdentity = 0;
    CIAttributeSliderMax = 1;
    CIAttributeSliderMin = 0;
    CIAttributeType = CIAttributeTypeScalar;
}, CIAttributeFilterName: CIColorMonochrome]
```

Wow, that’s a lot of filters! That should give you some ideas for other filters to try out in your own app!

哇，有很多的filters！它会给你一些灵感在你自己的app中尝试其他filters。

#More Intricate Filter Chains
#更加复杂的Filter链
Now that we’ve looked at all the filters that are available on the iOS platform, it’s time to create a more intricate filter chain. In order to do this, you’ll create a dedicated method to process the CIImage. It will take a CIImage, filter it to look like an old aged photo, and return a modified CIImage.

现在我们已经知道在iOS平台有哪些可用的filter，是时候创建更加复杂的filter链了。为了做到这样，你会创建一个专门的方法来处理CIImage。它会接受一个CIImage作为参数，过滤它，让它看起来像一张老旧的图片，然后返回一个已经修改的CIImage。

Add the following method to ViewController:

添加以下方法到ViewController：

```
func oldPhoto(img: CIImage, withAmount intensity: Float) -> CIImage {
  // 1
  let sepia = CIFilter(name:"CISepiaTone")
  sepia.setValue(img, forKey:kCIInputImageKey)
  sepia.setValue(intensity, forKey:"inputIntensity")
 
  // 2
  let random = CIFilter(name:"CIRandomGenerator")
 
  // 3
  let lighten = CIFilter(name:"CIColorControls")
  lighten.setValue(random.outputImage, forKey:kCIInputImageKey)
  lighten.setValue(1 - intensity, forKey:"inputBrightness")
  lighten.setValue(0, forKey:"inputSaturation")
 
  // 4
  let croppedImage = lighten.outputImage.imageByCroppingToRect(beginImage.extent())
 
  // 5
  let composite = CIFilter(name:"CIHardLightBlendMode")
  composite.setValue(sepia.outputImage, forKey:kCIInputImageKey)
  composite.setValue(croppedImage, forKey:kCIInputBackgroundImageKey)
 
  // 6
  let vignette = CIFilter(name:"CIVignette")
  vignette.setValue(composite.outputImage, forKey:kCIInputImageKey)
  vignette.setValue(intensity * 2, forKey:"inputIntensity")
  vignette.setValue(intensity * 30, forKey:"inputRadius")
 
  // 7
  return vignette.outputImage
}
```

Here’s what’s going on, section by section:

这里会逐段讲解：

1. Set up the sepia filter the same way you did in the simpler scenario. You’re passing in a Float value in the method to set the intensity of the sepia effect. This value will be provided by the slider.

1. 以相同的方式设置深褐色的filter。你在这个方法传递一个浮点数来设置深褐色效果的强度。这个值是由slider提供。

2. Set up a filter that creates a random noise pattern that looks like this:

2. 设置一个创建随机噪音模式的filter，filter效果如下：
<div align="center">
<img src="http://cdn4.raywenderlich.com/wp-content/uploads/2012/09/CIRandomGenerator.gif"/>
</div>

   It doesn’t take any parameters. You’ll use this noise pattern to add texture to your final “old photo” look.
   
   它好像没有接受任何参数。你会使用这个噪音模式来添加纹理到你的最终“老旧相片”的外观。

   
3. Alter the output of the random noise generator. You want to change it to grayscale, and lighten it up a little bit so the effect is less dramatic. You’ll notice that the input image key is set to the outputImage property of the random filter. This is a convenient way to pass the output of one filter as the input of the next.

3. 修改随机噪音生成器的输出。你想改变的它的灰度和减轻它一点以至于效果没有那么炫酷。你会注意到，输入image的键被设置到随机filter的outputImage属性。这是一种传递一个filter的输出作为下个输入的方便方式。

4. imageByCroppingToRect() takes an output CIImage and crops it to the provided rect. In this case, you need to crop the output of the CIRandomGenerator filter because it tiles infinitely. If you don’t crop it at some point, you’ll get an error saying that the filters have ‘an infinite extent’. CIImages don’t actually contain image data, they describe a ‘recipe’ for creating it. It’s not until you call a method on the CIContext that the data is actually processed.

4. **imageByCroppingToRect()**接受被裁剪的rect作为参数，然后返回CIImage。在这种情况下，你需要裁剪CIRandomGenerator filter的输出，因为它无限地碎片化。在某种情况下，如果你不裁剪它，你会得到一个错误信息，说filters有‘an infinite extent’。CIImages实际上是不包含image数据，它们创建它时将它描述成一个'recipe'。直到你在CIContext调用一个方法时数据才实际被处理。
 
5. Combine the output of the sepia filter with the output of the CIRandomGenerator filter. This filter performs the exact same operation as the ‘Hard Light’ setting does in a photoshop layer. Most (if not all) of the filter options in photoshop are achievable using Core Image.

5. 将深褐色filter的输出和CIRandomGenerator filter的输出结合起来。这个filter执行像在photoshop层‘Hard Light’设置完全一样的操作。大多数在photoshop(但不是全部)的filter选项都可以通过使用Core Image来实现。

6. Run a vignette filter on this composited output that darkens the edges of the photo. You’re using the value from the slider to set the radius and intensity of this effect.

6. 在这个合成的输出中，运行一个vignette filter，它会使你的图片边缘变暗。你使用从slider获取的值来设置半径和效果的强度。

7. Finally, return the output of the last filter.

7. 最后，返回最后filter的输出。

That’s all for this filter chain. You should now have an idea of how complex these filter chains may become. By combining Core Image filters into these kinds of chains, you can achieve an endless variety of effects.

关于filter chain讲到这里。你现在应该有一个如何形成复杂filter chain的想法。通过组合多个Core Image filters成各种各样的chains，你就能实现无穷多种的特效。

The next thing to do is implement this method in amountSliderValueChanged(). Change these two lines:

下一件事就是在**amountSliderValueChanged()**方法实现。改变这两行代码：

```
filter.setValue(sliderValue, forKey: "inputIntensity")
let outputImage = filter.outputImage
```

To this one line:

成这行代码：

```
let outputImage = self.oldPhoto(beginImage, withAmount: sliderValue)
```

This just replaces the previous sepia effect with your new, more complex filter method. You pass in the slider value for the intensity and you use the beginImage, which you set in the viewDidLoad method as the input CIImage. Build and run now and you should get a more refined old photo effect, complete with sepia, a little noise, and some vignetting.

这只是用新的，更复杂的filter方法来代替之前深褐色效果。你可以传递slder的值给强度，和你使用在viewDidLoad方法中被设置的beginImage作为输入的CIImage。编译和运行，你应该得到一个更加完善的旧图片效果，这种效果会是深褐色，有一点噪音和一点渐晕。

That noise could probably be more subtle, but I’ll leave that experiment to you, dear reader. Now you have the full power of Core Image at your disposal. Go crazy!

这种噪音效果会是更加微妙，但我会留给你自行试验。现在你完全有能力随意操纵Core Image，尽管去尝试吧！

#Where To Go From Here?（星期六）
#下一步

Here is the example project with all of the code from the above tutorial.

这个是[示例工程](http://cdn4.raywenderlich.com/wp-content/uploads/2014/07/CoreImageFun.zip)包含这个教程所有的代码。

That about covers the basics of using Core Image filters. It’s a pretty handy technique, and you should be able to use it to apply some neat filters to images quite quickly!

它包含了如何使用Core Image filters的基本方法。这是一个很容易上手的技术，所以你应该很快就能使用它来应用到一些优雅的filters。

Remember, the sample app prints out a list of all the available filters to the console when it starts up. Why not try playing around with some of those? You can check out the Core Image filter reference documentation for notes on all the filters.

记住，这个示例app在一开始的时候打印出所有可用的filters列表到控制台。为什么不尝试使用这些filters呢？你可以在[Core Image filter reference documentation](https://developer.apple.com/library/mac/documentation/GraphicsImaging/Reference/CoreImageFilterReference/Reference/reference.html)查看更多关于filters的详细信息。

If you have any questions or comments on this tutorial or Core Image in general, please join the forum discussion below!

如果你在这个教程中有任何问题或评论，请加入这个论坛来讨论。