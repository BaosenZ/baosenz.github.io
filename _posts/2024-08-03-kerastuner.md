---
title: "KerasTuner for Keras sequential neural network model hyperparameter tuning"
date: 2024-08-03 22:00:00 -0400
categories: [Tech, Machine Learning]
tags: [tech, machine learning]
image: https://upload.wikimedia.org/wikipedia/commons/thumb/a/ae/Keras_logo.svg/1024px-Keras_logo.svg.png
---

[KerasTuner](https://keras.io/keras_tuner/) was implemented to do hyperparameter tuning for Keras neural network (NN) sequential model. Using the code, I can tune the number of units, optimizer, activation function, and dropout rate. Then I can find the best model.  
KerasTuner auto the machine learning hyperparameter tuning, but it is still trial and error. So it takes a lot of computation time and it depends on our base model structure. Hopefully, I will have time to write more detailed guide for this. 

The dataset is from the paper, Liu, T., Johnson, K.R., Jansone-Popova, S. and Jiang, D.E., 2022. Advancing Rare-Earth Separation by Machine Learning. JACS Au, 2(6), pp.1428-1434. [https://pubs.acs.org/doi/full/10.1021/jacsau.2c00122](https://pubs.acs.org/doi/full/10.1021/jacsau.2c00122)

Dr. Joshua Schrier has his explorations. Check [his blog](https://www.wolframcloud.com/obj/jschrier0/Published/2023.11.30_towards_better_ML.nb).

More resources about KerasTuner are available online:   
[A YouTube video tutorial](https://www.youtube.com/watch?v=vvC15l4CY1Q)  
[The official keras_tuner doc](https://keras.io/guides/keras_tuner/getting_started/)

Below is the code for keras tuner. Full code is available here: <a href="https://colab.research.google.com/github/BaosenZ/code-repo-for-myblog/blob/master/python-learning/JacsAuPaper/KerasTuner_practice/JacsAu_paperCode_KerasTuner.ipynb" target="_blank"> <img alt="Open In Colab" src="https://colab.research.google.com/assets/colab-badge.svg" /></a>
```python
# build the model and define the search space using argument hp

def build_model(hp):
    model = keras.models.Sequential()

    # Input layer
    model.add(keras.layers.Input([2291]))
    
    # Define activation choice for all hidden layer
    activation_choice=hp.Choice("activation_all_layer", ["prelu", "relu", "tanh", "softmax"])

    # 1st layer
    if activation_choice == "prelu":
        model.add(keras.layers.Dense(
            # Tune number of units. 
            units=hp.Int("units", min_value=384, max_value=640, step=128),
            kernel_initializer='normal',
            # Set l2 weigh decay as 0.01
            kernel_regularizer=keras.regularizers.l2(0.01)
        ))
        model.add(keras.layers.PReLU())  # Add PReLU as a separate layer
    else:
        model.add(keras.layers.Dense(
            units=hp.Int("units", min_value=384, max_value=640, step=128),
            activation=activation_choice,
            kernel_initializer='normal',
            kernel_regularizer=keras.regularizers.l2(0.01)
        ))
    # Tune dropout rate.
    dropout_choise=hp.Choice("dropout_rate", [0.25, 0.5, 0.75])
    model.add(keras.layers.Dropout(rate=dropout_choise))
    
    # 2nd layer
    if activation_choice == "prelu":
        model.add(keras.layers.Dense(
            # Tune number of units. 
            128, 
            kernel_initializer='normal',
            # Set l2 weigh decay as 0.01
            kernel_regularizer=keras.regularizers.l2(0.01)
        ))
        model.add(keras.layers.PReLU())  # Add PReLU as a separate layer
    else:
        model.add(keras.layers.Dense(
            128, 
            activation=activation_choice,
            kernel_initializer='normal',
            kernel_regularizer=keras.regularizers.l2(0.01)
        ))
    # Tune dropout rate.
    model.add(keras.layers.Dropout(rate=dropout_choise))
    
    # 3rd layer
    if activation_choice == "prelu":
        model.add(keras.layers.Dense(
            # Tune number of units. 
            16,
            kernel_initializer='normal',
            # Set l2 weigh decay as 0.01
            kernel_regularizer=keras.regularizers.l2(0.01)
        ))
        model.add(keras.layers.PReLU())  # Add PReLU as a separate layer
    else:
        model.add(keras.layers.Dense(
            16,
            activation=activation_choice,
            kernel_initializer='normal',
            kernel_regularizer=keras.regularizers.l2(0.01)
        ))
    # Tune dropout rate.
    model.add(keras.layers.Dropout(rate=dropout_choise))
    
    # output layer
    model.add(keras.layers.Dense(1, kernel_initializer='normal'))
    
    # Compile the model
    # Define the optimizer choice and learning rate choise
    optimizer_choice = hp.Choice("optimizer", ["adam", "sgd", "rmsprop"])
    # lr_choice=hp.Choice("lr", values=[1e-6, 1e-5, 1e-4])
    if optimizer_choice == "adam":
        optimizer = keras.optimizers.Adam(
            learning_rate= 1e-5
        )
    elif optimizer_choice == "sgd":
        optimizer = keras.optimizers.SGD(
            learning_rate=1e-5
        )
    elif optimizer_choice == "rmsprop":
        optimizer = keras.optimizers.RMSprop(
            learning_rate=1e-5
        )
    
    model.compile(loss="mean_absolute_error", optimizer=optimizer, metrics=[keras.metrics.RootMeanSquaredError(), keras.metrics.R2Score()])

    return model
```

## Reference
1. Dr. Joshua Schrier blog: [https://www.wolframcloud.com/obj/jschrier0/Published/2023.11.30_towards_better_ML.nb](https://www.wolframcloud.com/obj/jschrier0/Published/2023.11.30_towards_better_ML.nb)
2. A YouTube video tutorial about KerasTuner: [https://www.youtube.com/watch?v=vvC15l4CY1Q](https://www.youtube.com/watch?v=vvC15l4CY1Q)
3. The official keras_tuner doc. Their doc is great: [https://keras.io/guides/keras_tuner/getting_started/](https://keras.io/guides/keras_tuner/getting_started/)
4. My code link: <a href="https://colab.research.google.com/github/BaosenZ/code-repo-for-myblog/blob/master/python-learning/JacsAuPaper/KerasTuner_practice/JacsAu_paperCode_KerasTuner.ipynb" target="_blank"> <img alt="Open In Colab" src="https://colab.research.google.com/assets/colab-badge.svg" /></a>


