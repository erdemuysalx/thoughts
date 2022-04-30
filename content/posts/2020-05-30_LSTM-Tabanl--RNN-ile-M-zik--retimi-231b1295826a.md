---
title: LSTM Tabanlı RNN ile Müzik Üretimi
description: >-
  Herkese selamlar. Bu yazımda sizlere LSTM (Long Short Term Memory) tabanlı
  Yinelemeli Sinir Ağları (RNN) ile müzik üretimi üzerine…
date: '2020-05-30T10:27:44.646Z'
categories: []
keywords: []
slug: /@erdemuysalx/lstm-tabanl%C4%B1-rnn-ile-m%C3%BCzik-%C3%BCretimi-231b1295826a
---

Herkese selamlar. Bu yazımda sizlere LSTM (Long Short Term Memory) tabanlı Yinelemeli Sinir Ağları (RNN) ile müzik üretimi üzerine bildiklerimi aktarmaya çalışacağım. Gelin öncelikle modeli geliştirmekte kullandığımız veri setinin temelde ne tür bilgiler içerdiğini irdeleyelim.

### Veri Seti Üzerine

Hepimizin de bildiği üzere müzikler notalar ile temsil edilmektedir. Ben veri seti olarak Do, Re, Mi tarzında klasik notaların yerine, ABC gösterimi olarak bilinen alternatif bir nota sistemini kullanmayı tercih ettim. ABC gösteriminde, verilen notaları temsil etmek için A’dan G’ye kadar olan harf gösterimleri ile beraber sayılar ve özel karakterler kullanılmaktadır. Her bir harf klasik notalarda bulunan Do, Re, Mi gibi klasik notalara karşılık gelmektedir. Melodiyi oluşturan notaların yanında ek bilgiler olarak diğer öğelerde kullanılmaktadır; referans numarası, besteci, köken, nota uzunluğu, tempo, ritim anahtar, süsleme vb. Bu değerlerin her biri veri setimizin özniteliklerini (features) oluşturmaktadır. ABC gösterimi genelde geleneksel halk müziklerini temsil etmek için kullanılmaktadır. Benim kullandığım veri seti de [İrlanda](https://cobb.ece.wisc.edu/irish/Tunebook.html) [Cobb](https://cobb.ece.wisc.edu/irish/Tunebook.html) [halk müziklerinde](https://cobb.ece.wisc.edu/irish/Tunebook.html)n¹ derlenmiş bir veri setidir. Aşağıda ABC gösterimi ile ifade edilen bir müzik örneği görmektesiniz.

X:1   
T:Alexander's   
Z: id:dc-hornpipe-1   
M:C|   
L:1/8   
K:D Major  
(3ABc|dAFA DFAd|fdcd FAdf|gfge fefd|(3efe (3dcB A2 (3ABc|! dAFA DFAd|fdcd FAdf|gfge fefd|(3efe dc d2:|! AG|FAdA FAdA|GBdB GBdB|Acec Acec|dfaf gecA|! FAdA FAdA|GBdB GBdB|Aceg fefd|(3efe dc d2:|!

Burada bir problem ile karşılaşmaktayız: bilgisayarlarımız harfleri ve özel karakterler içeren bir veriden anlam çıkarabilir mi? Öznitelikleri ağırlık (weights) değerleri ile çarpacağımızdan dolayı, B x 0.23 mantıksız bir işlem olacaktır, bu pek de mümkün görünmüyor öyle değil mi? İşte bu sebeple verileri bilgisayarın anlayabileceği bir formata getirmeliyiz. Tıpkı bilgisayarların resimleri piksel değerlerini içeren matrisler olarak temsil etmesi gibi,veri seti içerisinde bulunan her notaya, bir harfe veya bu özel karakter de olabilir, bir sayısal değer ataması gerçekleştireceğiz. Bu operasyona literatürde kodlama² (encoding) adı verilmektedir.

Kodlama işlemini gerçekleştirebilmek için öncelikle aşağıdaki kod parçası yardımıyla bütün şarkıları tek bir liste içerisinde topluyoruz. Bu liste içerisinde ki her bir özgün karakteri ayıklayıp küçükten büyüğe doğru sıraladıktan sonra `vocab` değişkenine atıyoruz. `vocab` değişkeni bizim şarkılarımızı temsil eden ve bilgi taşıyan eşsiz karakterleri içermektedir. Daha sonra bu eşsiz karakterlere nümerik değerleri atama yani kodlama işlemini gerçekleştirmemize izin veren: karakterleri sayılarla eşleyen `char2id` ve diğeri de sayıları karakterlerle eşleyen `idx2char` işlemlerini tanımlıyoruz.

Aşağıda, veriler içerisinde bulunan her bir karakteri ve o karaktere karşılık gelen nümerik değeri görmektesiniz.

{     
   '\\n':   0,  
   ' ' :   1,  
   '!' :   2,  
   '"' :   3,  
   '#' :   4,  
   "'" :   5,  
   '(' :   6,  
   ')' :   7,  
   ',' :   8,  
   '-' :   9,  
   '.' :  10,  
   '/' :  11,  
   '0' :  12,  
   '1' :  13,  
   '2' :  14,  
   '3' :  15,  
   '4' :  16,  
   '5' :  17,  
   '6' :  18,  
   '7' :  19,  
   ...   
}

Yukarıda bahsi geçen işlemleri daha düzenli olmaları açısında bir fonksiyon içerisinde bir araya getirelim.

Daha önce belirtilen atamalara göre karakter kodlama işlemi sonrasında harfler, sayılar ve özel karakterler içeren örnek verimiz aşağıda hale geliyor.

'X:1\\nT:Alex' <---kodlama işlemi---> \[49 22 13  0 45 22 26 67 60 79\]

Bu şekilde bir miktar veri ön işleme yaptıktan sonra artık veri setimizi kullanılabilir hale getirmiş oluyoruz.

Bir sonraki adımımız, şarkıları içeren metini asıl eğitim sırasında kullanacağımız örnek dizilere (batch) ayırmaktır. RNN’mizi beslediğimiz her girdi dizisi, metinden `seq_length` sayısı kadar karakter içermektedir. Ayrıca, bir sonraki karakteri tahmin etmek için RNN’nin eğitiminde kullanılmak üzere her bir giriş dizisi için ayrıca bir hedef dizi tanımlamamız gerekmektedir. Her giriş dizisi için, karşılık gelen hedef dizisi, bir karakter sağa kaydırılmış olması durumu dışında aynı uzunlukta karakter içerecektir.

Bunu yapmak için metni `seq_length + 1` sayısınca parçalara bölmeliyiz. `seq_length` değişkeninin 3 ve metnimizin “Ankara” olduğunu varsayarsak, girdi dizimiz “Ank” ve hedef dizi “nka” olacaktır.

Aşağıda tanımını görmekte olduğunuz fonksiyon, bu karakter dizisi akışını istenen boyuttaki dizilere dönüştürmemizi sağlamaktadır.

Deneme amaçlı oluşturulan batch üzerinden örnek girdi dizisi ve ardından beklenen değerler;

Step   0   input: 18 ('6')   expected output: 0 ('\\n')   
Step   1   input: 0 ('\\n')   expected output: 45 ('T')   
Step   2   input: 45 ('T')   expected output: 22 (':')   
Step   3   input: 22 (':')   expected output: 32 ('G')   
Step   4   input: 32 ('G')   expected output: 73 ('r')

Şimdi sıra modelimizi tasarlamakta. Modelimizde klasik Yapay Sinir Ağlarını kullanabilirdik, fakat müzik olarak adlandırdığımız melodiler, birbirini yineleyen bir dizi notalardan oluştuğu için, ileri besleme (feed-forward) tekniğini kullanan klasik Yapay Sinir Ağları bu problem için biraz işlevsiz kalıyor. Çünkü bizim müziği oluşturan notalar birbirinden bağımsız değil, tam aksine bir öncekine uyum sağlayacak şekilde birbiri ardına diziliyorlar. İşte bu yüzden modelimizde bu tarz dizi ve yineleme problemlerine çok güzel bir çözüm sağlayan ve daha gelişmiş bir yapay sinir ağı algoritması olan RNN yani Yinelemeli Sinir Ağlarını kullanacağız. Bu algoritma klasik sinir ağlarına ek olarak, girdi olarak bir önceki düğümün çıktısını alırken aynı zamanda paralelinde olan ve kendinden sonra gelen düğümlerinde çıktısını girdi olarak alabilmektedir. Aşağıda ileri beslemeli sinir ağları ile yinelemeli sinir ağlarının (RNN) farkını görebileceğiniz bir diyagramı bulabilirsiniz.

![](https://cdn-images-1.medium.com/max/800/1*edoo-1OI4uAIhi5wzD7u4A.png)

Her ne kadar bazı problemler için müthiş bir çözüm sunuyor olsada, RNN’lerinde bazı eksik noktaları var. Bunlardan bir tanesi, belli bir geçmiş noktaya kadar olan dizi elemanlarının bilgilerini sonra ki adımlara taşıyabiliyor olmaladır, bir diğer deyişle hafızaları oldukça kısa ve iş çok önceki elemanları kullanmaya geldiğinde algoritma zorlanmaya başlıyor. Buna çözüm olarak tam da bu tarz sorunları çözmeyi hedefleyen kapı hücreleri (gate cells) geliştirilmiştir. Bir kaç çeşit kapı hücresi tipi bulunmaktadır, örneğin LSTM (Long Short Term Memory) ve GRU. Biz bunlardan LSTM’i kullanacağız ve bunun detaylarına girmekten kaçınıyorum çünkü bir takım karmaşık matematiksel işlemler içeriyor ve şimdiki odak noktamız bunlar değil. Ama bir miktar önsezi olması açısından görselleştirilmiş bir LSTM kapı hücresini aşağıya bırakıyorum.

![](https://cdn-images-1.medium.com/max/800/1*cmv5EOAd6iWMzWvHrZbl-w.gif)

Modelimizin özetinin çıktısını aşağıda görmektesiniz.

Model: "sequential" \_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_ Layer (type)                 Output Shape              Param #    ================================================================= embedding (Embedding)        (32, None, 256)           21248      \_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_ lstm (LSTM)                  (32, None, 1024)          5246976    \_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_ dense (Dense)                (32, None, 83)            85075      ================================================================= Total params: 5,353,299 Trainable params: 5,353,299 Non-trainable params: 0 \_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_

Eğitim sona erdiğinde model bize kendi üretmiş olduğu müzik parçasını bir string dizisi olarak sunmaktadır. Bu diziyi kopyalayıp bilgisayarda açacağımız bir not defterine yapıştırıyor ve .abc uzantılı olarak kaydediyoruz. Bu işlemden sonra müzik parçasını dinleyebilirsiniz. Unutmayın, model her seferinde başarılı bir sonuç üretemeyebilir, hoş tınısı olan bir melodi için bir kaç kez üretim yapmak gerekebilir. İyi denemeler.

Eğitim sonunda ortaya çıkan eşsiz müzik parçaları:

Yazı çok uzayacağından dolayı kodların tamamını burada paylaşmak istemedim. Fakat herkesin deneyimleyebilmesi için veri seti ve noteboouku GitHub üzerinde paylaştım, aşağıdan ulaşabilirsiniz. Bir sonraki yazıda görüşmek üzere.

[**erd3muysal/Music\_Generation\_with\_LSTM\_Based\_RNN**  
_Contribute to erd3muysal/Music\_Generation\_with\_LSTM\_Based\_RNN development by creating an account on GitHub._github.com](https://github.com/erd3muysal/Music_Generation_with_LSTM_Based_RNN "https://github.com/erd3muysal/Music_Generation_with_LSTM_Based_RNN")[](https://github.com/erd3muysal/Music_Generation_with_LSTM_Based_RNN)

#### Kaynakça

\[1\] [https://cobb.ece.wisc.edu/irish/Tunebook.html](https://cobb.ece.wisc.edu/irish/Tunebook.html)

\[2\] [https://machinelearningmastery.com/how-to-prepare-categorical-data-for-deep-learning-in-python/](https://machinelearningmastery.com/how-to-prepare-categorical-data-for-deep-learning-in-python/)

\[3\] [http://introtodeeplearning.com/](http://introtodeeplearning.com/)

\[4\] [https://stanford.edu/~shervine/l/tr/teaching/cs-230/cheatsheet-recurrent-neural-networks](https://stanford.edu/~shervine/l/tr/teaching/cs-230/cheatsheet-recurrent-neural-networks)