---
editor_options: 
  markdown: 
    wrap: 72
---

# 第七讲：如何进行基本的数据分析: t-test和anova{#lesson-7}



## **语法实现**

### t检验
#### 理论基础
我们来讲讲t检验。最简单的是单样本t检验，它是一个样本的一系列数据，将其平均值与一个固定值进行比较。我们有两个假设，H0和H1，假设显著水平为0.05。NHST的逻辑是，首先假设H0为真，然后从H0统计模型中提取当前数据，计算概率。如果概率很低，我们认为它不太可能发生，拒绝假设H0。p值与假设水平进行比较时，我们假设H0为真。我们有三个选择，等于、大于或小于。如果不等于，那么是双样本检验，如果小于或大于，则为单样本检验。绍的是这些检验的不同类型。首先是单样本检验，当p值小于等于mu时，H0大于等于mu。另一种单样本检验是当H0大于等于mu时。

接下来是双样本检验，假设两个样本来自同一个集合。我们可以用p值或t值检验来比较两个样本之间的差异。如果数据是高度相关的，我们可以使用相关数据的t值检验。独立样本t值检验也是双样本检验，它是两对无相关的数据。我们可以使用R语言的函数来进行这些检验。有各种R语言函数可以用于t检验，这些函数包括在R-base基础包中。我们将会介绍Bruce R包，它是为心理学优化设计的，使用它更方便。
我们载入所需的包。BruceR包载入后会有一大串提示信息，包括载入的包、主要函数和网址等。我们使用相对路径读取之前的数据，可以查看前5行。
####解决问题

```r
library(bruceR)
```

```
## Warning: package 'bruceR' was built under R version 4.2.3
```

```
## 
## bruceR (v0.8.10)
## BRoadly Useful Convenient and Efficient R functions
## 
## Packages also loaded:
## ✔ data.table	✔ emmeans
## ✔ dplyr     	✔ lmerTest
## ✔ tidyr     	✔ effectsize
## ✔ stringr   	✔ performance
## ✔ ggplot2   	✔ interactions
## 
## Main functions of `bruceR`:
## cc()          	Describe() 	TTEST()
## add()         	Freq()     	MANOVA()
## .mean()       	Corr()     	EMMEANS()
## set.wd()      	Alpha()    	PROCESS()
## import()      	EFA()      	model_summary()
## print_table() 	CFA()      	lavaan_summary()
## 
## For full functionality, please install all dependencies:
## install.packages("bruceR", dep=TRUE)
## 
## Online documentation:
## https://psychbruce.github.io/bruceR
```

```
## 
## These packages are dependencies of `bruceR` but not installed:
## - cowplot, ggtext, lmtest, vars, phia, BayesFactor, GGally, GPArotation
## 
## ***** Install all dependencies *****
## install.packages("bruceR", dep=TRUE)
## ************************************
```

```r
library(dplyr)
library(tidyr)
```
1. “在penguin数据中，参与者对于家的依恋水平是否小于/等于/大于均值水平(假设在总体水平上，人们对家庭的依恋水平(HOME)均值为3.5)？”
2. “在penguin数据中，男女生在亲密关系经验(ECR)中的得分是否存在显著差异？”
3. “在penguin数据中，参与者在不同时间的温度差异是否具有显著性？”


```r
WD <-  here::here()

df.pg.raw <-  read.csv('./data/penguin/penguin_rawdata.csv',
                       header = T, sep=",", stringsAsFactors = FALSE)
```

首先，我们需要进行数据预处理，计算家庭依恋的均值。这需要使用之前学过的数据预处理函数和方法，我们将使用管道进行操作。为了计算第二个问题，我们首先筛选出两个性别，即男性和女性。然后，我们对数据进行操作，计算两个变量的得分均值，用mutate生成两个新变量。我们使用命令na.rm来去除包含缺失值的数据，以避免出现错误。我们可以使用命令将性别转换为factor，以便在进行t-test时更容易分类。在使用R语言时，不同的函数可能会有微小的区别，因此我们需要注意这些细节。在选择数据时，应该只选取感兴趣的变量进行分析，如年龄、性别、亲密关系、问卷得分和温度测量值等。

```r
df.pg.mean <- df.pg.raw %>%
  dplyr::filter(sex > 0 & sex < 3) %>%
  dplyr::mutate(ECR_mean = rowMeans(select(., starts_with("ECR")),na.rm = T),
                HOME_mean = rowMeans(select(., starts_with("HOME")),na.rm = T),
                sex=as.factor(sex)
                ) %>%
  dplyr::select(sex, ECR_mean, HOME_mean, Temperature_t1,Temperature_t2)
```


进行t-test时，需要输入数据框、y变量和x变量，同时需要注意是否是配对样本和是否满足方向性。

```r
  bruceR::TTEST(df.pg.mean, "HOME_mean", test.value = 3.5, test.sided = ">")
```

```
## 
## One-Sample t-test
## 
## Hypothesis: one-sided (μ > 3.5)
```

```
## `BayesFactor` package needs to be installed.
```

```
## Descriptives:
## ───────────────────────────
##   Variable    N Mean (S.D.)
## ───────────────────────────
##  HOME_mean 1490 3.99 (0.73)
## ───────────────────────────
## 
## Results of t-test:
## ──────────────────────────────────────────────────────────────────────────────────────────────
##                                   t   df     p     Difference [95% CI] Cohen’s d [95% CI] BF10
## ──────────────────────────────────────────────────────────────────────────────────────────────
## HOME_mean: (HOME_mean - 3.5)  25.97 1489 <.001 ***    0.49 [0.46, Inf]   0.67 [0.63, Inf]     
## ──────────────────────────────────────────────────────────────────────────────────────────────
```

```r
  bruceR::TTEST(df.pg.mean, "HOME_mean", test.value = 3.5, test.sided = "<")
```

```
## 
## One-Sample t-test
## 
## Hypothesis: one-sided (μ < 3.5)
```

```
## `BayesFactor` package needs to be installed.
```

```
## Descriptives:
## ───────────────────────────
##   Variable    N Mean (S.D.)
## ───────────────────────────
##  HOME_mean 1490 3.99 (0.73)
## ───────────────────────────
## 
## Results of t-test:
## ──────────────────────────────────────────────────────────────────────────────────────────────
##                                   t   df     p     Difference [95% CI] Cohen’s d [95% CI] BF10
## ──────────────────────────────────────────────────────────────────────────────────────────────
## HOME_mean: (HOME_mean - 3.5)  25.97 1489 1.000       0.49 [-Inf, 0.52]  0.67 [-Inf, 0.72]     
## ──────────────────────────────────────────────────────────────────────────────────────────────
```

```r
  result.ttest <- capture.output({
  bruceR::TTEST(data=df.pg.mean, 
                y="HOME_mean", 
                test.value = 3.5, 
                test.sided = "=",
                file = "./output/chp7/single_t.doc")
})
```

```
## `BayesFactor` package needs to be installed.
```

```r
writeLines(result.ttest, "./output/chp7/single_t.md") # .md最整齐
```


mean difference代表两组数据的均值差异，是t-test的一个效应量。test side表示双维还是单维，。test value是要检验的变量。factor variance表示是否对因素进行反转，digital参数指小数点后保留的位数，file参数用于保存结果到word文档。
使用capture output output函数可以整理结果并复制到results.ttest变量中。将结果复制到名为“results.ttest”的变量中，并将结果写入一个名为“single_t.md”的文件中。在根目录中打开R4SAC项目文件，我们可以看到已经运行过的输出结果，其中包括一个符合心理学格式的三线表格，其中包含了均值、t值、自由度、p值、协方差和效应量等信息。这个表格非常干净.

```r
    result.ttest <- capture.output({   
    bruceR::TTEST(data=df.pg.mean, 
                y="ECR_mean",
                x="sex")
})
```

```
## `BayesFactor` package needs to be installed.
```

```r
writeLines(result.ttest, "./output/chp7/inde_t.md")
```
然后是独立样本t检验，比较男性和女性之间的差异。使用了ECRmin作为样本得分，用性别作为分组变量。结果是two-side的，有描述性的结果，结果包括每组的n、均值和方差齐性检验。如果方差不齐性，可以使用校正方法。t检验的结，包括t值、df值、p值、mean difference

```r
   result.ttest <- capture.output({
   bruceR::TTEST(data=df.pg.mean, 
                 y = c("Temperature_t1",              
                       "Temperature_t2"),
                 paired = T)  #是否为配对样本t检验？默认是FALSE
})
```

```
## `BayesFactor` package needs to be installed.
```

```r
writeLines(result.ttest, "./output/chp7/pair_t.md")
```
配对样本t检验也很简单，只需要将y变成两个值，然后使用t1和t2进行检验。

请大家注意，我们之前讲过如何管理和放置你的R-Project，可以避免路径错误。如果你不熟悉路径，可以从github上下载整个文件夹，打开R4psy文件夹中地R-Project文件。我们的助教已经在群里分享了GitHub链接。打开链接后，点击绿色按钮下面的“code”，再点击“download zip”，就可以下载整个文件夹。下载后解压缩，把文件夹放在一个完整的文件夹里。文件夹里有一个叫做“R4psy”的文件，双击它就可以了。
有时候打开课程内容，运行代码会看到一个三角或加号，加号表示代码没有输完，可能是缺少反括号或不知道加号前面跟哪个对应。可以补上反括号或按退出键退出。Rstudio是一个好的IDE，可以帮助我们导航代码。在Rmarkdown里，代码块以chunk1、chunk2等命名，可以快速定位到代码框。有些chunk没有命名，只有编号。也可以通过outline来定位。
如果你想要快速运行代码，推荐使用快捷键control+回车，在Mac上是command+回车。也可以通过查找keyboard shortcut来找到更多快捷键。

在进行统计分析后，可以使用软件包reports来标准化报告结果，包括均值、标准差、t值、p值和Cohen's d等。

### 方差分析

我们将使用Brusa和Tidyburst软件包，但需要注意，安装BaseFact包可能会出现错误，因为它依赖于Jax包。如果出现错误，可以在官方网站上搜索如何安装BaseFact包。注意安装依赖包，否则可能会出现问题。在使用Pinode时，可能会遇到一些警告和问题，需要自己解决。

```r
df.match <-  read.csv('./data/match/match_raw.csv',
                      header = T, sep=",", stringsAsFactors = FALSE) %>%
  # 拆分单元格内字符串
  tidyr::separate(col=Shape,
                  into=c("Valence","Identity"),
                  sep="(?<=moral|immoral)(?=Self|Other)") %>% 
  dplyr::select(Sub, Valence, Identity, everything()) %>%
  dplyr::filter(ACC == 1) %>%    # 只选择回答正确的数据
  dplyr::filter(!is.na(RT)) %>%  # 剔除缺失值
  # remove outliers below and above 3rd sd
  dplyr::filter(RT > quantile(RT, 0.0015) & RT < quantile(RT, 0.9985)) %>%
  dplyr::mutate(RT = as.numeric(RT)) %>%
  dplyr::mutate(Valence = as.factor(Valence),
                Identity = as.factor(Identity))
```


```r
WD <-  here::here()

df.match.raw <-  read.csv('./data/match/match_raw.csv',
                       header = T, sep=",", stringsAsFactors = FALSE)%>%
                 tidyr::separate(., col=Shape,into = c("Valence","Identity"),
                                              sep = "(?<=moral|immoral)(?=Self|Other)")%>% #拆分单元格内字符串
                 dplyr::select(Sub,Valence,Identity,everything())%>%
                 dplyr::filter(ACC == 1) %>% #只选择回答正确的数据
                 dplyr::filter(RT > quantile(RT, 0.0015) & RT < quantile(RT, 0.9985)) %>% #remove outliers below and above 3rd sd
                 dplyr::mutate(RT = as.numeric(RT)) %>%
                 dplyr::mutate(across("ACC", as.factor)) %>%
                 dplyr::mutate(Valence = as.factor(Valence)) %>%
                 dplyr::mutate(Identity = as.factor(Identity)) %>%
                 dplyr::filter(!is.na(RT)) #剔除缺失值
```

```r
DT::datatable(head(df.match.raw, 10),
              fillContainer = TRUE, options = list(pageLength = 5))
```

```{=html}
<div id="htmlwidget-57b371e62a35ab1f63f7" style="width:100%;height:auto;" class="datatables html-widget"></div>
<script type="application/json" data-for="htmlwidget-57b371e62a35ab1f63f7">{"x":{"filter":"none","vertical":false,"fillContainer":true,"data":[["1","2","3","4","5","6","7","8","9","10"],[7302,7302,7302,7302,7302,7302,7302,7302,7302,7302],["moral","immoral","immoral","immoral","moral","moral","moral","immoral","moral","immoral"],["Other","Other","Self","Self","Other","Self","Other","Self","Other","Other"],["02-May-2018_14:23:08","02-May-2018_14:23:10","02-May-2018_14:23:15","02-May-2018_14:23:17","02-May-2018_14:23:19","02-May-2018_14:23:21","02-May-2018_14:23:24","02-May-2018_14:23:26","02-May-2018_14:23:28","02-May-2018_14:23:30"],["Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp"],[22,22,22,22,22,22,22,22,22,22],["female","female","female","female","female","female","female","female","female","female"],["R","R","R","R","R","R","R","R","R","R"],[1,1,1,1,1,1,1,1,1,1],[1,1,1,1,1,1,1,1,1,1],[2,3,5,6,7,8,9,10,11,12],["moralOther","immoralOther","immoralSelf","immoralSelf","moralOther","moralSelf","moralOther","immoralSelf","moralOther","immoralOther"],["mismatch","mismatch","match","match","match","match","mismatch","mismatch","mismatch","match"],["n","n","m","m","m","m","n","n","n","m"],["n","n","m","m","m","m","n","n","n","m"],["1","1","1","1","1","1","1","1","1","1"],[0.7043,0.9903,0.8207,0.7547,0.5429,0.9009,0.9551,0.6952,0.7593,0.7135]],"container":"<table class=\"display fill-container\">\n  <thead>\n    <tr>\n      <th> <\/th>\n      <th>Sub<\/th>\n      <th>Valence<\/th>\n      <th>Identity<\/th>\n      <th>Date<\/th>\n      <th>Prac<\/th>\n      <th>Age<\/th>\n      <th>Sex<\/th>\n      <th>Hand<\/th>\n      <th>Block<\/th>\n      <th>Bin<\/th>\n      <th>Trial<\/th>\n      <th>Label<\/th>\n      <th>Match<\/th>\n      <th>CorrResp<\/th>\n      <th>Resp<\/th>\n      <th>ACC<\/th>\n      <th>RT<\/th>\n    <\/tr>\n  <\/thead>\n<\/table>","options":{"pageLength":5,"columnDefs":[{"className":"dt-right","targets":[1,6,9,10,11,17]},{"orderable":false,"targets":0}],"order":[],"autoWidth":false,"orderClasses":false,"lengthMenu":[5,10,25,50,100]}},"evals":[],"jsHooks":[]}</script>
```
在数据清理时，可以先读取数据，然后进行管道操作，将Shape列分割成Valence和Identity两列，并选择Base、Valence、Identity和Everything列。然后进行数据清理，只选择反应正确的反应时间，并去掉缺失值和极端反应时间。我们使用了三个标准差的方法来去除极端值。我们将反应时间转换为数值型，将情感和身份转换为因子型。我们将所有操作应用于数据框df.match，这是经过预处理的数据。

```r
df.match.mean <- df.match.raw %>%
    dplyr::group_by(Sub, Match) %>%
    dplyr::summarise(
        n = n(),
        rt_sd = sd(as.numeric(RT), na.rm = T),
        rt_mean = mean(as.numeric(RT), na.rm = T),
        acc_mean=mean(as.numeric(as.character(ACC)),na.rm=T)#对于factor数据，先转成character，再变成numeric
        ) %>%
    dplyr::ungroup()
```

```
## `summarise()` has grouped output by 'Sub'. You can override using the `.groups`
## argument.
```
我们需要求出每个被试在匹配和不匹配任务上的均值，才能进行比较。为此，我们使用分组进行预处理，分组的目标是为了得到每个组的描述性统计，如RT的mean和SD，以及正确试次的个数。我们可以使用summarize快速得出这些统计信息。接着，我们可以使用T-test或方差分析来比较匹配和不匹配反应是否有差异。


```r
#比较被试在匹配任务和不匹配任务上的反应时是否存在差异
#单因素被试内设计（长型数据）
result.anova <- capture.output({
df_within <- df.match.mean%>%
  bruceR::MANOVA(subID="Sub",#被试id
                 dv="rt_mean",#因变量，因为是MANOVA，实际上因变量可以再后面设置很多个
                 within="Match") #设置条件，因素分析的条件
})
```

```
## 
##     * Data are aggregated to mean (across items/trials)
##     if there are >=2 observations per subject and cell.
##     You may use Linear Mixed Model to analyze the data,
##     e.g., with subjects and items as level-2 clusters.
```

```r
writeLines(result.anova, "./output/chp7/anova_1.md")
# 若不符合球形假设要加上：sph.correction = "GG"
```

在方差分析中，我们可以使用BruceR包中的MANOVA()函数。参数分别是subID,dv和within，即可直接对其进行操作并输出结果。这几个参数符合心理学的命名规范。我们可以得到match和mismatch两种条件的描述性统计，包括f值、p值、eta2、pash2、95%置信区间和generalized eta2。

因为这是一个完全被试内设计的分析，所以不需要进行方差齐性检验。

```r
df.match.mean <- df.match %>%
    dplyr::group_by(Sub,Sex, Valence,Identity) %>%
    dplyr::summarise(
        n = n(),
        rt_sd = sd(as.numeric(RT), na.rm = T),
        rt_mean = mean(as.numeric(RT), na.rm = T)
        ) %>%
    dplyr::ungroup()
```

```
## `summarise()` has grouped output by 'Sub', 'Sex', 'Valence'. You can override
## using the `.groups` argument.
```
对于被试身份和效价这两个因素，我们可以直接从match中选出数值对，然后进行groupby和summarize操作。需要注意的是，我们现在是以subject、variance和identity为基础进行分析。

```r
df.match.within <- df.match.mean %>%
  dplyr::select(-c(n, rt_sd)) %>%
  dplyr::mutate(Valence = paste("A_", Valence, sep = ""),#将变量的名称进行修改转换，不生产新的变量
                Identity = paste("B_", Identity, sep = "")) %>%
  # 将morality和identity组合名称起来生成一个新的变量"Conds"
  tidyr::unite("Conds", Valence:Identity, sep = "&",remove=TRUE) %>% 
  tidyr::pivot_wider(names_from = Conds,
                     values_from = rt_mean)

head(df.match.within)
```

```
## # A tibble: 6 × 6
##     Sub Sex    `A_immoral&B_Other` `A_immoral&B_Self` `A_moral&B_Other` A_mora…¹
##   <int> <chr>                <dbl>              <dbl>             <dbl>    <dbl>
## 1  7302 female               0.706              0.724             0.662    0.721
## 2  7303 male                 0.741              0.754             0.720    0.700
## 3  7304 female               0.756              0.730             0.757    0.628
## 4  7305 male                 0.685              0.659             0.641    0.638
## 5  7306 male                 0.778              0.763             0.660    0.771
## 6  7307 female               0.751              0.677             0.716    0.728
## # … with abbreviated variable name ¹​`A_moral&B_Self`
```

```r
#str(df.match.within)
```

```r
#被试在不同身份(self vs other)与不同效价(moral vs moral)的条件组合下反应时是否存在差异）
  result.anova <- capture.output({
  res_rmANOVA_1 <- bruceR::MANOVA(data=df.match.within,
                   dvs="A_immoral&B_Other:A_moral&B_Self",
                   dvs.pattern="A_(.+)B_(.+)",#正则表达式
                   within=c("A_","B_"))#表示2个主条件
  })
```

```
## 
## Note:
## dvs="A_immoral&B_Other:A_moral&B_Self" is matched to variables:
## A_immoral&B_Other, A_immoral&B_Self, A_moral&B_Other, A_moral&B_Self
```

```r
writeLines(result.anova, "./output/chp7/anova_2.md")
```
如果我们有的是宽数据，我们可以先将宽数据转成长数据，然后选择变量并按照Bruce R的命名方式在前面加上a和b表示自变量a和自变量b。接着，我们可以使用unite函数将Valence和Identity合并成一个新的变量，再使用pivot wide将其转换为更宽的形式。这样，我们就可以看到每个条件下的性别、被试id和条件。
最后，我们将结果写成md格式，得到一个简洁的表格，其中包含f值、p值、partial square、generalized square和partial square的95%置信区间。长数据也可以使用同样的方法处理。

```r
#被试在不同身份(self vs other)与不同效价(good vs bad)的条件组合下反应时是否存在差异）
#head(df.match.mean)
  result.anova <- capture.output({
  res_rmANOVA_2 <- bruceR::MANOVA(data=df.match.mean,
                   dv="rt_mean",
                   within=c("Valence","Identity"),
                   subID="Sub")
  })
```

```
## 
##     * Data are aggregated to mean (across items/trials)
##     if there are >=2 observations per subject and cell.
##     You may use Linear Mixed Model to analyze the data,
##     e.g., with subjects and items as level-2 clusters.
```

```r
writeLines(result.anova, "./output/chp7/anova_3.md")
```

```r
result.check <- capture.output({
sim_eff_1 <- res_rmANOVA_1 %>%
  bruceR::EMMEANS("A_", by="B_")#简单效应分析
})
writeLines(result.check, "./output/chp7/check.md")

#sim_eff_1 <- res_rmANOVA_1 %>%
  #EMMEANS("B_", by="A_")   和上一个同理，只是转换了不同的形式
```
除了基础分析方法，我们还可以使用EMMEANS进行简单效应分析和多重比较矫正等操作。通过输入之前的方差分析结果，我们可以使用EMMEANS a by b来进行简单效应分析，以比较在不同条件下的主效应是否存在差异。
我们要分析a在b的不同条件下的效应，包括other和self条件。在other条件下，a的效应是b条件的一个主效应，我们称之为简单效应。我们可以看到，在b的两个条件下，a的简单效应实际上应该有一个交互作用。在other条件下，它不显著，在self条件下，它很显著。
此外，我们还可以看到a的quantity，即在self条件下，a的主效应moral和immoral之间差异非常大，quantity大于0.8，是一个大的效应。
