# DETR_Segmentation

## Contents
- [Objective](#objective)
- [Approach](#approach)
- [References](#references) 

------------------------------

### Approach

Since ViT (an image is worth 16x16 words) works very effective in object classification as compared to CNN methods in virtue of computer vision. But to achieve an object detection, segmentation, panoptic segmentation, object localization, object colorization, we need the help of convolutions to achieve. So Detr or Detection transformers was developed by Facebook Team.

DeTR utilizes Resnet50 architecture to perform the convolutions. When we have an RGB input image **3 x H x W**, it is passed to the backbone model (Resnet50) and is restricted till conv5_x (we don't perform average pool, fc & softmax), hence we arrive **2048** channels and **H/32, W/32** (32 here refers to the time we scale down the image in the Resnet architecture), which would have captured all the possible features of the image. Finally we arrive at **2048 x H/32 x W/32**, but 2048 channels would be a huge data going in , so we pass this through a simple FC layer which reduces the number of channels to say **256 x H/32 x W/32** or **d x H/32 x W/32** (Here we keep copy of this ouput aside, which would be useful in the later stage). 


![image](https://user-images.githubusercontent.com/47082769/131225487-0f2cc3c8-1c8c-4fc5-ae0e-c613181b31fc.png)


Then this is passed to a Encoder-Decoder block (Nx - number of blocks in encoder & decoder would be same) where in the encoder we would be having Self attention for keys & values & in decoder queries would be fed & it goes through self attention & also cross attention  (where keys, values  comes from encoder & it interacts with the query from the decoder). Finally we arrive at the encodded image **d x H/32 x W/32** where we would be having object embeddings, **taking the below image as an example**, the object embeddings would be - for the foreground cow and an embedding for the each segment of the background namely the sky, grass and trees. We then use a **multihead attenttion** layer that returns the attention scores for the encoded image for each object embedding which would be the attention maps - **N x M x H/32 x W/32**, where N is the number of object embeddings.


![image](https://user-images.githubusercontent.com/47082769/131226878-90452ee2-0d81-4f8c-b7d4-ee362bb4bc45.png)

We later proceed to upsample the activation maps and clean these masks using a convolutional network that uses the intermediate activations from the backbone (these intermediate activations are the copy of output from the Resnet block, which we kept from the intial step). As a result we get high resoulution maps where each pixel contains binary logic belonging to the mask. Finally we merge the masks by assigning each pixel to the mask with the highest logit a simple pixel wise.


### References

- https://arxiv.org/pdf/2005.12872.pdf
- https://www.youtube.com/watch?v=utxbUlo9CyY






