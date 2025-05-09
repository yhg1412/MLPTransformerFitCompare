# MLPTransformerFitCompare
This repository compares small size MultiLayer Perceptron(MLP), Transformer and Convolutional Neural Network(CNN) architecture on predicting sine values.

We find that Transformer performs better than MLP and CNN. In addition, when auto regressivly generating data, transformer generated data shows a fixed periodic pattern irrelevant to data initialization.

## Introduction

Transformer[1] is a widely adopted machine learning architecture which contributed to the success of Large Language Models(LLM) like BERT and GPTs. These language models perform well on many language tasks.

Theoretically, Yun et al.[2] proved that Transfomer can approximate any continuous sequence-to-sequence function in a compact domain. Later, different variants of transformers are proposed. Some of them using relative positional encoding are proved not universal approximators while others are.[3]

Multilayer Perceptron(MLP) is also proved to be an universal approximator with one hidden layer given unbounded width of the network[4] or bounded width but unbounded depth of the network[5]. CNN is able to approximate any continuous function to an arbitrary accuracy when the depth of the neural network is large enough.[6]

Theorectical analysis shows both MLP and Transformer are capable of approximating arbitary functions in a compact domain. However, it often assumes unbounded network size. This repository experiment on MLP and Transformer predicting sine values. It trys to answer the following question: 

With similar number of parameters(#params) and similar number of operations(#FLOPS), MLP and Transformer, which one performs better when approximating time series data generated by function sin(wt)

It is similar to time series prediction if we only look at one of the output channel. It's worth mentioning that, whether Transformer is good for time series forecasting is still being debated. In 2020, Zhou et al.[7] introduced Informer which is a transformer based model for time series forecasting. In 2022, Zeng et al.[8] published their results showing that Transformers are not effective for time series forecasting compared to DLinear model. However, in 2023, Hugging face published their findings[9] showing an opposite result: on Traffic and Electricity Dataset, Autoformer performs better.

## Data

Data are generated with 100 sine functions with different frequencies. Frequency range is [-5, 5].

There are 800 timestamps sampled ranging from -20 to 20. The sampling interval is 0.05.

50 hisotoric timestamp values are used to predict the current timestamp data.

Each input data entry shape is 100*50: each row corresponds to one frequency and consists of 50 historic sine values.

<img width="1461" alt="DataExample" src="https://github.com/yhg1412/MLPTransformerFitCompare/assets/17675483/d11324d7-d550-4124-90ea-a1861b2bad45">

## Model Setup and Training

Transformer
  - Number of attention heads: 4
  - Number of decoder layers: 4
  - dmodel: 100
  - feedforward layer dimension: 100
  - number of trainable parameters: 254,100

Multilayer Perceptron(MLP)
  - 3 layers: 5000 X 400 X 400 X 100
  - number of trainable parameters: 2,200,900 

Convolutional Neural Network(CNN) 
  - Two convolusional layers with maxpool: [7*7, 32], [7*7, 64]
  - Two feedforward layers: [10240, 200], [200, 100]
  - number of trainable parameters: 2,170,316

Training
  - loss function: MSELoss
  - optimizer: SGD
  - learning rate: Transformer 0.2, MLP 0.01, CNN 0.01
  - epochs: Transformer 40, MLP 60, CNN 40

## Results

Mean Square Error Loss
  - Baseline: 0.00040
  - Transformer: 0.00011
  - MLP: 0.00029
  - CNN: 0.00019

For Baseline, we use x[t-1], x[t-2] to predict x[t] = 2x[t-1] - x[t-2].

Transformer performed best and the loss is calculated based on the last row of Transformer output only.

## Findings

We let the trained models to autoregressively generate data and observe the graph. We plot one output dimension and find out that the curve generated by Transformer is showing a periodic pattern as can be seen in below figure.

<img width="740" alt="TSR5F1" src="https://github.com/yhg1412/MLPTransformerFitCompare/assets/17675483/c17c2629-8669-48e3-a82c-af5cc6bb6839">

~~MLP converged to a constant value as can be seen from below figure.~~ (Wrong)

<img width="753" alt="MLPH2R5F1" src="https://github.com/yhg1412/MLPTransformerFitCompare/assets/17675483/07d9a48a-aed7-4b18-8761-884744241cdd">

2025-04-18 Update: MLP might not "converge"(looks like a constant) to a constant for some frequencies:

<img width="745" alt="MLP_generated_freq70" src="https://github.com/user-attachments/assets/503d2a86-7a38-4369-a4ab-908dd3985608" />

CNN displayed a different shape of periodic curve.

<img width="747" alt="CNNR5F1" src="https://github.com/yhg1412/MLPTransformerFitCompare/assets/17675483/966ae2f9-8a24-4b67-b827-8db32330d9ce">


We further experimented what parameters could influence its amplitude and period. The following figure shows that the period is irrelevant to output channel frequencies and has a positve correlation with training data frequency. In the below graph, first column is trained from lower frequency input data and second column is trained from higher frequency data. We can see the generated data frequency is also higher in the second column. In addition, by comparing each rows we can see that different output channels show same frequency so the generated data frequency is irrelevant with output channel frequency.

<img width="794" alt="Screen Shot 2024-04-16 at 11 23 37 AM" src="https://github.com/yhg1412/MLPTransformerFitCompare/assets/17675483/0436d5b5-9a1e-45d2-948e-b023d51a5533">

In addition, we also use random initialized data as input to let the trained model to generate data autogressively. We find that Transformer still show a periodic pattern as can be seen from below graph.

<img width="749" alt="TSRandomInputR5F1" src="https://github.com/yhg1412/MLPTransformerFitCompare/assets/17675483/15a1d53e-99d6-4dad-9a74-54ddf4b9cb27">


## ~~Conclusion~~
## Observation

~~In this experiment, Transformer performs better than MLP and CNN in predicting generated sine data.~~ 

In this experiment, in terms of loss, transformer performs better than MLP for this model setup.


We also find that when auto-regressivly generate data using trained models, a periodic signal is observed. Further mathamatical study may be needed to explain it. One possible explaination to this is related to logistic map.[10] For equation X_n+1 = rX_n(1 - X_n), when r is in certain ranges like slightly beyond 3.54409, from almost all initial conditions of X_0, X_n will approach periodic oscillations among 8 values, then 16, 32, etc. In Transformer, self-attention mechanism will also compute a quadratic term of historical values similar to the above equation.

## References

[1] Vaswani, Ashish et al. “Attention is All you Need.” Neural Information Processing Systems (2017).

[2] Yun, Chulhee et al. “Are Transformers universal approximators of sequence-to-sequence functions?” International Conference on Learning Representations (2019).

[3] Luo, Shengjie et al. “Your Transformer May Not be as Powerful as You Expect.” Neural Information Processing Systems (2022).

[4] Hornik, Kurt. “Approximation capabilities of multilayer feedforward networks.” Neural Networks 4 (1991): 251-257.

[5] Lu, Zhou et al. “The Expressive Power of Neural Networks: A View from the Width.” Neural Information Processing Systems (2017).

[6] Zhou, Ding-Xuan. “Universality of Deep Convolutional Neural Networks.” Applied and Computational Harmonic Analysis (2018).

[7] Zhou, Haoyi et al. “Informer: Beyond Efficient Transformer for Long Sequence Time-Series Forecasting.” AAAI Conference on Artificial Intelligence (2020).

[8] Zeng, Ailing et al. “Are Transformers Effective for Time Series Forecasting?” AAAI Conference on Artificial Intelligence (2022).

[9] Eli, Simhayev et al. "Yes, Transformers are Effective for Time Series Forecasting (+ Autoformer)" (https://huggingface.co/blog/autoformer) (2023)

[10] Logistic Map. (https://en.wikipedia.org/wiki/Logistic_map)
