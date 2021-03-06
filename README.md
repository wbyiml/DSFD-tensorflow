# dsfd

## introduction

A tensorflow implement dsfd, and there is something different with the origin paper.

It‘s a ssd-like object detect framework, but slightly different,
combines lots of tricks for face detection, such as dual-shot, dense anchor match, FPN,FEM and so on.

now it is mainly optimised about face detection,
and borrows some codes from other repos

ps, the code maybe not that clear, please be patience, and i am still working on it, and forgive me for my poor english :)


the evaluation results are based on vgg with batchsize(2x6),pretrained model can be download from
https://pan.baidu.com/s/1cUqnf9BwUVkCy0iT6EczKA ( password ty4d )

widerface  val set

| Easy MAP | Medium MAP | hard MAP |
| :------: | :------: | :------: |
|  0.942 | 0.935 | 0.880 |


| fddb   |
| :------: | 
|  0.987 | 



## requirment

+ tensorflow1.12

+ tensorpack (for data provider)

+ opencv

+ python 3.6

## useage

### train
1. download widerface data from http://shuoyang1213.me/WIDERFACE/
and release the WIDER_train, WIDER_val and wider_face_split into ./WIDER, then run
```python prepare_wider_data.py```it will produce train.txt and val.txt
(if u like train u own data, u should prepare the data like this:
`...../9_Press_Conference_Press_Conference_9_659.jpg| 483(xmin),195(ymin),735(xmax),543(ymax),1(class) ......` 
one line for one pic, **caution! class should start from 1, 0 means bg**)
2. download the imagenet pretrained vgg16 model from http://download.tensorflow.org/models/vgg_16_2016_08_28.tar.gz
release it in the root dir,

3. but if u want to train from scratch set config.MODEL.pretrained_model=None,

4. if recover from a complet pretrained model  set config.MODEL.pretrained_model='yourmodel.ckpt',config.MODEL.continue_train=True

then, run:

`python train.py`

and if u want to check the data when training, u could set vis in train_config.py as True


#### ** CAUTION， WHEN USE TENSORPACK FOR DATA PROVIDER, some change is needed. **
#### in lib/python3.6/site-packages/tensorpack/dataflow/raw.py ,line 71-96. to make the iterator unstoppable, change it as below. so that we can keep trainning when the iter was over. contact me if u have problem about the codes : )
```
 71 class DataFromList(RNGDataFlow):
 72     """ Wrap a list of datapoints to a DataFlow"""
 73 
 74     def __init__(self, lst, shuffle=True):
 75         """
 76         Args:
 77             lst (list): input list. Each element is a datapoint.
 78             shuffle (bool): shuffle data.
 79         """
 80         super(DataFromList, self).__init__()
 81         self.lst = lst
 82         self.shuffle = shuffle
 83     
 84     #def __len__(self):
 85     #    return len(self.lst)
 86 
 87     def __iter__(self):
 88         if not self.shuffle:
 89             for k in self.lst:
 90                 yield k
 91         else:
 92             while True:
 93                 idxs = np.arange(len(self.lst))
 94                 self.rng.shuffle(idxs)
 95                 for k in idxs:
 96                     yield self.lst[k]
```



### evaluation

```
    python model_eval/fddb.py [--model [TRAINED_MODEL]] [--data_dir [DATA_DIR]]
                          [--split_dir [SPLIT_DIR]] [--result [RESULT_DIR]]
    --model              Path of the saved model,default ./model/detector.pb
    --data_dir           Path of fddb all images
    --split_dir          Path of fddb folds
    --result             Path to save fddb results
 ```
    
example `python model_eval/fddb.py --model model/detector.pb 
                                    --data_dir 'fddb/img/' 
                                    --split_dir fddb/FDDB-folds/ 
                                    --result 'result/' `

```
    python model_eval/wider.py [--model [TRAINED_MODEL]] [--data_dir [DATA_DIR]]
                           [--result [RESULT_DIR]]
    --model              Path of the saved model,default ./model/detector.pb
    --data_dir           Path of WIDER
    --result             Path to save WIDERface results
 ```
example `python model_eval/wider.py --model model/detector.pb 
                                    --data_dir 'WIDER/WIDER_val/' 
                                    --result 'result/' `


### visualization
![A demo](https://github.com/610265158/dsfd_tensorflow/blob/master/res_screenshot_11.05.2019.png)

(caution: i dont know where the demo picture coms from, if u think it's a tort, i would like to delet it)

if u get a trained model, run `python tools/auto_freeze.py`, it will read the checkpoint file in ./model, and produce detector.pb, then

`python vis.py`

u can check th code in vis.py to make it runable, it's simple.




### details
#### anchor

if u like to show the anchor stratergy, u could simply run :

`python anchor/utils.py`


it will draw the anchor one by one,


#### data augmentation

if u like to know how the data augmentation works, run :

`python data/augmentor/augmentation.py`
