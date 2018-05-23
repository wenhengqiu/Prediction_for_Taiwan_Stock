# Prediction_for_Taiwan_Stock

## <center>股票之預測</center>

- @author: Vincent Yau
- @version: 1.0 | 2018/05/21
- @email: wenhengqiu@gmail.com

### 数据前处理
- 从一堆数据中获得与台股2912相关的文章，分为2016年及2017年
- 需要解决数据中NaN的问题，尤其是用正则匹配中文字后，数据为空的情况
- 我选择的处理方法：
    - 1、对DataFrame中的NaN，采用dropna()方法
    - 2、对于分词中的空值，赋予一个生字string（不是一个好的方法，但因为我们选好了特征词，所以一个字符对结果影响不大[说明还是有影响]）
    - 2、对于terms的tfidf的缺失，采用赋予0.0001（不是一个好方法）

### 程式逻辑整理
- 讀取某一年的所有文章，獲得所有的日期，存為 dateList
- 獲得某一天的所有文章，每篇文章都作為list的元素保存起來 articlesList，然後直接存入{日期:articlesLsit}字典中
    - 這裡要注意的是，盡量對數據只進行一次讀取，一次讀取多次使用，避免因為頻繁讀取造成了資源浪費
- 對articlesList中的每一篇文章進行分詞處理，單獨計算其 Term Frequency，保存成 Dictionary
    - 使用jieba分詞，而不用N-gram是為了提高分詞的準確性，且可針對經濟新聞定制Stop Words列表
    - 採用jieba之前，對傳入的characters進行正則匹配，獲取其中所有的中文
    - 中文正則表達式為[^\u4e00-\u9fa5]，故當string無中文時會產生NaN，解決方法參考數據前處理
- 計算某一天所有文章詞的tf-idf，整合成一個dict保存起來
- 獲得特征詞表（1000個詞）
- 將每天的文章按照 [特征詞 + tfidf]進行表示，將所有的天數的數據都append到一個List中

### 特征詞的獲得
- 將所有的文章，根據股票的漲跌判斷，分為「看漲文章」「看跌文章」
- 分別將兩類文章進行Features Selection（特征詞抽取），這裡抽取的依據是TF-IDF，目測抽取1000個詞
- 注意特征詞必須要有順序

### 股票漲跌判斷
- 若第x天的股價，與x+n天的股價存在一定比例的漲or跌，則將x天的文章列為漲or跌文章（估計我們會分為漲、跌、不漲不跌）
- Actually，這裡直接將漲跌的label與日期對應起來即可
- 漲跌的判斷在Machine learning中，屬於對數據進行標標籤，因此，我們的模型屬於：監督式學習
- 理論上，若將每篇文章都進行此標記，則會增Traning Dataset，但因數據文章有許多comments，如此操作會造成大量的稀疏數據
- 將某天的所有文章，用list合併一起，并最終用「特征詞：tfidf」進行表示，是現今採用的方式

### 分類器的構建
- MultinomialNB
- Sppport Vector Machine
- GaussianNB
