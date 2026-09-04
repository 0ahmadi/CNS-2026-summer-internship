**Can a neural network trained on the same visual stimuli as mice predict the neural representations of those stimuli in the mouse visual cortex, and how does this similarity vary across visual cortical areas and cortical depths?**

## Dataset

We use the Allen Institute Neuropixels dataset (session 757216464). Mice passively view 118 natural scene images while a probe records spiking activity across four visual cortical areas: LGd, VISp, VISrl, VISam.

## What we're doing

The basic idea is simple: show the same 118 images to both a mouse and a neural network, then check if their internal representations look alike.

- Neural network: AlexNet, pretrained on ImageNet (not trained on our images)
- Brain data: firing rates recorded from each of the 4 areas for each image

If a network layer and a brain area respond to images in a similar way (similar images get similar responses, different images get different responses), we say they're "aligned."

## Step 1: RSA (Representational Similarity Analysis)

For every AlexNet layer and every brain area, we build a matrix showing how different each pair of images looks to that layer/area (a 118x118 dissimilarity matrix). Then we compare these matrices across layers and areas using correlation. This tells us which layer of AlexNet best matches which brain area.

<img width="2986" height="1479" alt="image" src="https://github.com/user-attachments/assets/cde99e63-b634-4d36-a9c6-c2cfbea8a1ff" />

<img width="2763" height="1272" alt="image" src="https://github.com/user-attachments/assets/0dff77ce-bfe0-426e-bdc0-b722fe14f1d2" />


## Step 2: NeuroCLIP

RSA only measures similarity with a fixed formula, it can't learn anything. So we built a small CLIP-style model: two encoders, one for the network side and one for the brain side, trained with a contrastive loss so that the same image's network embedding and brain embedding end up close together, and different images end up far apart.

We do this separately for every layer/area pair (8 layers x 4 areas = 32 models), then compare validation loss and retrieval accuracy across all of them, same idea as the RSA heatmap but this time based on something the model actually learned instead of a fixed statistic.

Because we only have 118 images, we added two kinds of data augmentation to fight overfitting:
- On the brain side: resample a random real trial (not just the trial-averaged response) each epoch
- On the network side: feed the model a randomly noise-corrupted version of the image each epoch (sparse noise only, no rotation/cropping, since those would actually change what the mouse's brain does, and we don't have that data)

<img width="2672" height="1470" alt="image" src="https://github.com/user-attachments/assets/0587e74c-ea68-4bb8-b86c-fe9256968bc3" />

## Where things stand

- All 4 brain areas show statistically significant similarity to AlexNet, but the effect sizes are modest
- The best-matching layer isn't always the "expected" one for each area, no clean hierarchy
- NeuroCLIP lets us test alignment in a learned way instead of a fixed formula, and gives us a reusable architecture, not just a one-off result for this dataset
