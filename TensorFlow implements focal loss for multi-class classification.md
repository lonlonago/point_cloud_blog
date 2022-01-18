TensorFlow implements focal loss for multi-class classification


Because the classification of the classification data is unbalanced and serious recently, I considered replacing the original loss, but I found several versions of the focal loss implementation code on the Internet, either the final result is not right, or it does not fully meet my needs, So I simply rewrote the code of one of them myself, and the record is as follows.



There are 3 concepts that need to be clarified as follows:

When calculating loss, the input is Logits or the probability of prob is two different things. Strictly speaking, the final output of a model has three types, logits, prob, prediction, the first one is the original output of the network, which is (N, Class) dimension Yes, the second is the output probability distribution after softmax, which is also of (N, Class) dimension, and the third is the output after argmax, which is N single-dimensional. And many codes do not give the part of softmax.



Multi-category classification and multi-label classification are different. One final result is a unique category label, and the other is that categories can exist at the same time, and their result data dimensions are also different. This is only for multi-category classification.



Input size, usually the input size of logits is (N, Class), and if the input size of labels is after one hot, it is also (N, Class), otherwise it is (N,), and this is not clear in many codes. In addition, TensorFlow supports input logits whose size exceeds 2 dimensions, such as (N, X, Class) can also be calculated.


The focal loss in his code has no alpha weight, I added it;

There is a bug in the code, and suggestions for modification are given;

Added the handling of probability 0;

In addition, the input size is also modified if it is not (N, Class) but more dimensions;

The size of each variable is also annotated;

code show as below:

````python

# 注意，alpha是一个和你的分类类别数量相等的向量；
alpha=[[1], [1], [1], [1]]

def focal_loss(logits, labels, alpha，epsilon = 1.e-7,
                   gamma=2.0, 
                   multi_dim = False):
        '''
        :param logits:  [batch_size, n_class]
        :param labels: [batch_size]  not one-hot !!!
        :return: -alpha*(1-y)^r * log(y)
        它是在哪实现 1- y 的？ 通过gather选择的就是1-p,而不是通过计算实现的；
        logits soft max之后是多个类别的概率，也就是二分类时候的1-P和P；多分类的时候不是1-p了；

        怎么把alpha的权重加上去？
        通过gather把alpha选择后变成batch长度，同时达到了选择和维度变换的目的

        是否需要对logits转换后的概率值进行限制？
        需要的，避免极端情况的影响

        针对输入是 (N，P，C )和  (N，P)怎么处理？
        先把他转换为和常规的一样形状，（N*P，C） 和 （N*P,）

        bug:
        ValueError: Cannot convert an unknown Dimension to a Tensor: ?
        因为输入的尺寸有时是未知的，导致了该bug,如果batchsize是确定的，可以直接修改为batchsize

        '''


        if multi_dim:
            logits = tf.reshape(logits, [-1, logits.shape[2]])
            labels = tf.reshape(labels, [-1])

        # (Class ,1)
        alpha = tf.constant(alpha, dtype=tf.float32)

        labels = tf.cast(labels, dtype=tf.int32)
        logits = tf.cast(logits, tf.float32)
        # (N,Class) > N*Class
        softmax = tf.reshape(tf.nn.softmax(logits), [-1])  # [batch_size * n_class]
        # (N,) > (N,) ,但是数值变换了，变成了每个label在N*Class中的位置
        labels_shift = tf.range(0, logits.shape[0]) * logits.shape[1] + labels
        #labels_shift = tf.range(0, batch_size*32) * logits.shape[1] + labels
        # (N*Class,) > (N,)
        prob = tf.gather(softmax, labels_shift)
        # 预防预测概率值为0的情况  ; (N,)
        prob = tf.clip_by_value(prob, epsilon, 1. - epsilon)
        # (Class ,1) > (N,)
        alpha_choice = tf.gather(alpha, labels)
        # (N,) > (N,)
        weight = tf.pow(tf.subtract(1., prob), gamma)
        weight = tf.multiply(alpha_choice, weight)
        # (N,) > 1
        loss = -tf.reduce_mean(tf.multiply(weight, tf.log(prob)))
        return loss

````


The method of determining the value of alpha is reversed according to the sample scale:




After verification, the code is consistent with the conventional softmax_cross_entropy_with_logits calculation results when alpha is set to 1 and gama=0.


questions to contract me : lonlonago@foxmail.com
