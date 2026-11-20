# Neural network
 Spectral separation network consists of two different neural networks:
*	Forward model: Halftone to spectra
*	Backward model: spectra to halftone

We should start by training the forward network first (HalftoneToSpec.py) then we freeze the weights of the forward and train the backward model in an encoder-decoder like architecture (Forward+Backward.py).

Once we have trained the general backward model we start the adaptive learning (Adaptive learning.py), This will improve the loss of reproduction.

Once the adaptive model is trained we can generate the halftone area coverage of the painting (generate_halftone.py).

The codes are tuned for reproducing the painting of the flower but we can load other paintings from the “Mixed-integer-ink-selection/Dataset/Spectral paintings/*.mat”
Due to the large size of the images, we have to process them in smaller batches and attach them together later (to avoid exceeding the memory of GPU). 

The output of (generate_halftone.py) is a series of *.npy files containing the halftone of the painting in an n x 8 structure which we need to reshape later (\Prepare the area coverage to print\coverage2haftone.m) to get our 8 layer halftone maps. Each for one ink channel in the printer. 

In order to test the effect of changing the network capacity on this task, we have trained a smaller network (small_forward_net.py) with 2 hidden layers each 50 perceptrons wide. The difference in the loss is insignificant, as a result, we can replace this smaller network for the tasks where smaller capacity is an advantage.

In addition to this 8 Epson ink network, you can find another network (Canon_Net.py). This is the forward model trained on a set of 4 Canon inks.
