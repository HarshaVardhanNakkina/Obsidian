## Convolutions

So, what's convolution? You might ask. Well, if you've ever done any kind of image processing, it usually involves having a filter and passing that filter over the image in order to change the underlying image. The process works a little bit like this. For every pixel, take its value, and take a look at the value of its neighbors. If our filter is three by three, then we can take a look at the immediate neighbor, so that you have a corresponding three by three grid. Then to get the new value for  he pixel, we simply multiply each neighbor by the corresponding value in the filter.

The idea here is that some convolutions will change the image in such a way that certain features in the image get emphasized. So, for example, if you look at this  filter, then the vertical lines in the image really pop out. With this filter, the horizontal lines pop out.

Now, that's a very basic introduction to what convolutions do, and when combined with something called pooling, they can become really powerful. But simply, pooling is a way of compressing an image. A quick and easy way to do this, is to go over the image of four pixels at a time, i.e., the current pixel and its neighbors underneath and to the right of it. Of these four, pick the biggest value and keep just that.

![[Pasted image 20230428111352.png]]
![[Pasted image 20230428152535.png]]
___
## Max pooling

![[Pasted image 20230428153610.png]]

take max of each region
this is as if applying a filter of 2x2 and a stride of 2
these are the **hyper parameters** of max pooling

![[Pasted image 20230428153825.png]]

![[Pasted image 20230428154516.png]]
___
## Average pooling

instead of taking max, take average
![[Pasted image 20230428154651.png]]

### summary of pooling
Hyperparameters:
- f: filter size
- s: stride
Max or Average pooling
- p: mostly 0 in pooling
we don't pad in pooling mostly
___
## Neural Network example (LeNet-5)

![[Pasted image 20230428155618.png]]

![[Pasted image 20230428155807.png]]

- \#parameters = f\*f + \<bias parameter\> + no. of filters
___
## Why convolutions

![[Pasted image 20230428163032.png]]

