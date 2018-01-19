## TensorFlow দিয়ে ইমেজ ক্লাসিফিকেশনের উপযোগী একটি NN তৈরি  
এই সেকশনে আমরা, অনেক রকম মানুষের বিভিন্ন রকম হাতের লেখা ওয়ালা কিছু নাম্বার/ডিজিট এর ফটো কালেকশন দিয়ে একটা নিউরাল নেটওয়ার্ক-কে ট্রেইন করিয়ে তারপর কিছু টেস্ট ফটো দিয়ে সেগুলোর সঠিক ক্লাসিফিকেশন জানার চেষ্টা করবো। সহজ ভাবে বলতে - "হ্যান্ড রিটেন ডিজিট ক্লাসিফিকেশন প্রব্লেম"।

এই টিউটোরিয়ালে আমরা কনভলিউশনাল নিউরাল নেটওয়ার্ক মডেল ব্যবহার <span style="text-decoration:underline;">করছি না</span>। বরং সিঙ্গেল লেয়ারের লিনিয়ার মডেল ব্যবহার করবো। অর্থাৎ একটি ইনপুট লেয়ার এবং একটি আউটপুট লেয়ার থাকবে, কিন্তু ইনপুট লেয়ারে প্রথম দিকের উদাহরণ এর মত কয়েকটি না বরং অনেক গুলো নিউরন থাকবে। আর আউটপুট লেয়ারে থাকবে ১০টি নিউরন। ১০ ধরনের ডিজিট ক্লাসিফিকেশনের জন্য। এতে করে আমাদের TensorFlow দিয়ে কাজ করার কমন কিছু স্টেপ সম্বন্ধে পরিষ্কার ধারনা আসবে। তবে হ্যাঁ, এই লিনিয়ার ক্লাসিফায়ারও যথেষ্ট ভালো মতই ডিজিট ক্লাসিফিকেশন করতে পারবে আশা করা যায়। অন্তত ৮৫-৯০% সঠিক ক্লাসিফাই করতে পারবে।
এই টিউটোরিয়ালের পর আমরা একই সমস্যা আরও ইফেক্টিভ ভাবে সমাধানের জন্য এবং অ্যাকিউরেসি লেভেল আরও বাড়ানোর জন্য কনভলিউশনাল নিউরাল নেটওয়ার্ক মডেল ব্যবহার করবো। যা হোক, এখন আমরা ধাপে ধাপে সব গুলো কাজ করবো এবং লেয়ার তৈরি করবো এবং তার মাঝে মাঝেই কিছু নতুন টার্ম আসবে, সেগুলোর প্রয়োজনীয়তা এবং সংজ্ঞা জানবো।</p>
প্রথমেই আমরা কিছু রেডিমেড ডাটা সেট নিয়ে কাজ করবো। অর্থাৎ ডিজিট ক্লাসিফিকেশন শিখতে গিয়ে আমাদের অনেক অনেক ইমেজ দরকার পরবে আমাদের মডেলকে ট্রেনিং দেয়ার জন্য, তাই না? এমনকি ট্রেনিং ডাটাগুলো লেবেল্ড (কোন ফটো কোন ডিজিট তার একটা ম্যাপিং) হতে হবে। নাহলে ট্রেনিং হবে ক্যামনে? এখন নিউরাল নেটওয়ার্ক শিখতে গিয়ে যদি মাসের পর মাস সময় দিয়ে শুধু ডাটাই রেডি করতে হয় তাহলে ক্যামনে কি?
মজার বিষয় হচ্ছে, TensorFlow -এর সাথেই এরকম কিছু রেডিমেড ডাটা থাকে এবং যেগুলো চাইলে আমরা import করে সেগুলোর উপর কাজ করতে পারি। অন্তত আসল জিনিষ শেখার সময় আমাদের পুরো সময়টা ডাটা প্রি-প্রসেসিং -এ নষ্ট হচ্ছে না। ফোকাস থাকবে মডেল ডেভেলপমেন্টে। যা হোক, এই ডাটাবেজটার নাম হচ্ছে <a href="http://yann.lecun.com/exdb/mnist/">MNIST</a> ডাটাবেজ/ডাটাসেট।

এই টিউটোরিয়ালের জন্য আমরা <a href="http://jupyter.org/">Jupyter Notebook</a> ব্যবহার করবো। এতে করে ধাপে ধাপে আলোচনা করে করে আগানো যাবে এবং আগের ধাপে রান করা কোড পরের ধাপেও অ্যাক্সেস করা যাবে। jupyter Notebook সম্পর্কে ধারনা না থাকলে একটু অন্য কোথাও থেকে আপাতত দেখে আসতে পারেন। এটা তেমন কিছু না। একটা ওয়েব অ্যাপ। এতে করে ব্রাইজারের মধ্যে একটা পেজে কোড এবং বাংলা ইংলিশ মিলিয়ে লেখা যায় এবং কোড গুলোকে রানও করা যায়। আর পুরো ডকুমেন্টের রানটাইম একটাই থাকে। এটাও খুব সহজে প্যাকেজ আকারেই ইন্সটল করা যায় এবং একটা কমান্ড দিয়েই রান করানো যায়। কথা না বাড়িয়ে শুরু করা যাক।

যদিও শেষের দিকে পুরো প্রোগ্রামের একটা স্ক্রিপ্ট ভার্সন থাকবে যেটা স্বাভাবিকভাবে নোটবুকের বাইরেও রান করানো যাবে।

```python
# Cell 1
%matplotlib inline
import matplotlib.pyplot as plt
import tensorflow as tf
import numpy as np
from sklearn.metrics import confusion_matrix
```

উপরের ব্লক নিয়ে কিছু বলার দরকার আছে কি? সব চেনা জিনিষ, একটা বাদে। confusion_matrix ব্যবহার করে আমরা একধরনের স্পেশাল ম্যাট্রিক্স তৈরি ও ডিসপ্লে করতে পারি যার মাধ্যমে আমরা কিছু রিলেটেড এরর এর বৈশিষ্ট্য সম্পর্কে একটা ভিজুয়াল ধারনা পাবো। এটার স্টেপ আসা মাত্রই এর দরকারটাও বোঝা যাবে। যদিও এটা অপশনাল স্টেপ। আমাদের মুল মডেল তৈরিতে এটার গুরুত্ব নাই, বরং Accuracy বাড়াতে এবং সমস্যার উপর একটা স্পষ্ট ধারনা আনতে সাহায্য করবে এই ম্যাট্রিক্স। অর্থাৎ, সমস্যা নিয়ে ভালো অব্জারভেশন করতে চাইলে এগুলো লাগে। আরেকটা জিনিষ - %matplotlib inline যার মাধ্যমে জুপিটার নোটবুকের চলতি ডকুমেন্টটির মধ্যেই প্লটিং গুলো ডিসপ্লে করার কথা বলা হচ্ছে। তো নোটবুকের প্রথম সেলে এই কোড লিখে সেলটি এক্সিকিউট করে ফেলি।

এরপর আমাদের ডাটাগুলোকে লোড করতে হবে, এর জন্য নিচের কোড টুকু ব্যবহার করতে পারি অর্থাৎ পরের সেলে লিখে সেলটি এক্সিকিউট করতে পারি,

```python
# Cell 2
from tensorflow.examples.tutorials.mnist import input_data
data = input_data.read_data_sets("data/MNIST/", one_hot=True)
```

এর মাধ্যমে ১২ মেগাবাইট সাইজের ডাটাসেটটি ডাউনলোড হবে যদি data/MNIST/ পাথে আগে থেকেই ডাটাসেটটি না থাকে।