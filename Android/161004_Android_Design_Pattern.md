

#Journey for developing MapSee mobile Application#


----------


##1. Introduction##

간단한 모바일 어플리케이션 개발에도,  서버 관리 지식과 관련 API 사용, 그리고 안드로이드 전반에 관한 지식이 필요하다.  이 문서에서는 여행 어플리케이션(MapSee) 개발 초기부터 배포 과정에서의 경험을 체계적으로 정리하고 공유함으로써, 여러 사람들에게 유익한 정보를 제공하고자 한다. 

----------


    
##2. MVP model ##


 어플리케이션은 여러 기능들의 유기적인 집합이라고 볼 수 있다.  이런 기능들의 구현을 적절히 분배하는 것은 협업 프로세스를 체계화하여 일관성을 확보하고, 코드 내 문제 발견과 해결 그리고 추가 기능 구현을 효율화하는데 매우 핵심적인 요소이다.

이와 같은 분배를 효율적으로 하기 위해서는, 기능 단위로 책임을 분배하고 각 부분에서 월권행위를 하지 않을 수 있도록 여러 장치를 마련해 주는 것이 좋다.

    
이러한 효율적인 분배에 대한 설계의 노하우가 축적된 것이 MVC, MVVM, MVP 과 같은 **디자인패턴**[^designPattern]인데, 우리는 MapSee 어플리케이션의 구현을 위해 MVP를 이용할 것이다.




####**2.1)  MVP 패턴이란? **

프로그램이 동작하는 과정에서, 프로그램은 user-interface 상에서 유저의 커맨드와 input을 입력받고, 이를 저장하거나 처리한 뒤 다시 user 에게 결과를 알려준다.
이러한 동작 프로세스를 크게 세가지 부분으로 분리할 수 있는데,   MVP가 제안하는 분리는 바로 **모델, 뷰, 프레젠터**이다.  이 파트에서는 각각에 대해서 보다 개념적으로 알아보도록 하자.  이 부분에서 각 개념들이 쉽게 이해되지 않는다고 걱정하지 말자. 이후 회원 가입 안드로이드 프로그램 예제를 통해 이 개념들을 더욱 쉽게 이해 할 수 있도록 할 것이다. 이 부분에서는 각 부분의 역할들을 한 번 읽어보는 것으로 충분하다.

> 1)    **모델 (Model)**
> 
> : 모델은 유저 인터페이스를 구현하는 데 참고할 데이터를 정의하는 인터페이스이다. (_The model is an interface defining the data to be displayed or otherwise acted upon in the user interface._) 쉽게 풀어쓰자면, 모델은 유저의 정보를 포함한 다양한 정보들을 실세계의 로직에 따라 관리하기 위한 프로그램의 한 모듈이라고 생각해도 좋다.    
> 
> 2)   **뷰(View)**
>
> : 뷰는 모델을 보여주고, 유저에 의해 발생한 이벤트들을 프레젠터에게 라우팅 시켜주는 인터페이스이다. (_The view is a passive interface that displays data (the model) and routes user commands (events) to the presenter to act upon that data._) 이 역시 쉽게 풀어쓰자면, 뷰는 스마트폰에서 우리가 터치하거나 키보드를 통해 글을 입력하는 UI를 제공하고, 여기서 이벤트(터치, 혹은 글자 변동)가 발생했을 때 프레젠터에게 이벤트 발생을 알려주는 역할을 하는 모듈이다.
>
>3) **프레젠터(Presenter)**

>: 프레젠터는 모델과 뷰에 따라 행동하며, 모델에서 데이터를 인출하여 뷰에게  이를 전달함으로써, UI를 변화시키도록 지시한다. (_presenter acts upon the model and the view. It retrieves data from repositories (the model), and formats it for display in the view._) 즉, 프레젠터는 모델과 뷰 사이에 있는 중간 관리인이라고 할 수 있으며, 정보와 커맨드를 적절히 상호 간에 전달한다.
>
> ![enter image description here](https://upload.wikimedia.org/wikipedia/commons/d/dc/Model_View_Presenter_GUI_Design_Pattern.png)
> 영문으로 된 내용은 Model–view–presenter, Wikipedia 에서 발췌한 것이다.[^MVP]

이 개념들을 읽다보면 자연스럽게 드는 의문은 'Model, View, Presenter 는 안드로이드 코드에서 어떤 부분에 해당하는가?'일 것이다. View가 Activity 인지, Model 이 하나의 Class를 말하는 것인지 이제 회원가입 안드로이드 프로그램 예제를 통해 MVP 디자인 패턴을 이해해보도록 하자.

####**2.2) MVP 패턴 예제  - 회원 가입 프로그램**

Documents
-------------

StackEdit stores your documents in your browser, which means all your documents are automatically saved locally and are accessible **offline!**

> **Note:**

> - StackEdit is accessible offline after the application has been loaded for the first time.
> - Your local documents are not shared between different browsers or computers.
> - Clearing your browser's data may **delete all your local documents!** Make sure your documents are synchronized with **Google Drive** or **Dropbox** (check out the [<i class="icon-refresh"></i> Synchronization](#synchronization) section).

#### <i class="icon-file"></i> Create a document

The document panel is accessible using the <i class="icon-folder-open"></i> button in the navigation bar. You can create a new document by clicking <i class="icon-file"></i> **New document** in the document panel.

#### <i class="icon-folder-open"></i> Switch to another document

All your local documents are listed in the document panel. You can switch from one to another by clicking a document in the list or you can toggle documents using <kbd>Ctrl+[</kbd> and <kbd>Ctrl+]</kbd>.

#### <i class="icon-pencil"></i> Rename a document

You can rename the current document by clicking the document title in the navigation bar.

#### <i class="icon-trash"></i> Delete a document

You can delete the current document by clicking <i class="icon-trash"></i> **Delete document** in the document panel.

#### <i class="icon-hdd"></i> Export a document

You can save the current document to a file by clicking <i class="icon-hdd"></i> **Export to disk** from the <i class="icon-provider-stackedit"></i> menu panel.

> **Tip:** Check out the [<i class="icon-upload"></i> Publish a document](#publish-a-document) section for a description of the different output formats.


----------


Synchronization
-------------------

StackEdit can be combined with <i class="icon-provider-gdrive"></i> **Google Drive** and <i class="icon-provider-dropbox"></i> **Dropbox** to have your documents saved in the *Cloud*. The synchronization mechanism takes care of uploading your modifications or downloading the latest version of your documents.

> **Note:**

> - Full access to **Google Drive** or **Dropbox** is required to be able to import any document in StackEdit. Permission restrictions can be configured in the settings.
> - Imported documents are downloaded in your browser and are not transmitted to a server.
> - If you experience problems saving your documents on Google Drive, check and optionally disable browser extensions, such as Disconnect.

#### <i class="icon-refresh"></i> Open a document

You can open a document from <i class="icon-provider-gdrive"></i> **Google Drive** or the <i class="icon-provider-dropbox"></i> **Dropbox** by opening the <i class="icon-refresh"></i> **Synchronize** sub-menu and by clicking **Open from...**. Once opened, any modification in your document will be automatically synchronized with the file in your **Google Drive** / **Dropbox** account.

#### <i class="icon-refresh"></i> Save a document

You can save any document by opening the <i class="icon-refresh"></i> **Synchronize** sub-menu and by clicking **Save on...**. Even if your document is already synchronized with **Google Drive** or **Dropbox**, you can export it to a another location. StackEdit can synchronize one document with multiple locations and accounts.

#### <i class="icon-refresh"></i> Synchronize a document

Once your document is linked to a <i class="icon-provider-gdrive"></i> **Google Drive** or a <i class="icon-provider-dropbox"></i> **Dropbox** file, StackEdit will periodically (every 3 minutes) synchronize it by downloading/uploading any modification. A merge will be performed if necessary and conflicts will be detected.

If you just have modified your document and you want to force the synchronization, click the <i class="icon-refresh"></i> button in the navigation bar.

> **Note:** The <i class="icon-refresh"></i> button is disabled when you have no document to synchronize.

#### <i class="icon-refresh"></i> Manage document synchronization

Since one document can be synchronized with multiple locations, you can list and manage synchronized locations by clicking <i class="icon-refresh"></i> **Manage synchronization** in the <i class="icon-refresh"></i> **Synchronize** sub-menu. This will let you remove synchronization locations that are associated to your document.

> **Note:** If you delete the file from **Google Drive** or from **Dropbox**, the document will no longer be synchronized with that location.

----------


Publication
-------------

Once you are happy with your document, you can publish it on different websites directly from StackEdit. As for now, StackEdit can publish on **Blogger**, **Dropbox**, **Gist**, **GitHub**, **Google Drive**, **Tumblr**, **WordPress** and on any SSH server.

#### <i class="icon-upload"></i> Publish a document

You can publish your document by opening the <i class="icon-upload"></i> **Publish** sub-menu and by choosing a website. In the dialog box, you can choose the publication format:

- Markdown, to publish the Markdown text on a website that can interpret it (**GitHub** for instance),
- HTML, to publish the document converted into HTML (on a blog for example),
- Template, to have a full control of the output.

> **Note:** The default template is a simple webpage wrapping your document in HTML format. You can customize it in the **Advanced** tab of the <i class="icon-cog"></i> **Settings** dialog.

#### <i class="icon-upload"></i> Update a publication

After publishing, StackEdit will keep your document linked to that publication which makes it easy for you to update it. Once you have modified your document and you want to update your publication, click on the <i class="icon-upload"></i> button in the navigation bar.

> **Note:** The <i class="icon-upload"></i> button is disabled when your document has not been published yet.

#### <i class="icon-upload"></i> Manage document publication

Since one document can be published on multiple locations, you can list and manage publish locations by clicking <i class="icon-upload"></i> **Manage publication** in the <i class="icon-provider-stackedit"></i> menu panel. This will let you remove publication locations that are associated to your document.

> **Note:** If the file has been removed from the website or the blog, the document will no longer be published on that location.

----------


Markdown Extra
--------------------

StackEdit supports **Markdown Extra**, which extends **Markdown** syntax with some nice features.

> **Tip:** You can disable any **Markdown Extra** feature in the **Extensions** tab of the <i class="icon-cog"></i> **Settings** dialog.

> **Note:** You can find more information about **Markdown** syntax [here][2] and **Markdown Extra** extension [here][3].


### Tables

**Markdown Extra** has a special syntax for tables:

Item     | Value
-------- | ---
Computer | $1600
Phone    | $12
Pipe     | $1

You can specify column alignment with one or two colons:

| Item     | Value | Qty   |
| :------- | ----: | :---: |
| Computer | $1600 |  5    |
| Phone    | $12   |  12   |
| Pipe     | $1    |  234  |


### Definition Lists

**Markdown Extra** has a special syntax for definition lists too:

Term 1
Term 2
:   Definition A
:   Definition B

Term 3

:   Definition C

:   Definition D

	> part of definition D


### Fenced code blocks

GitHub's fenced code blocks are also supported with **Highlight.js** syntax highlighting:

```
// Foo
var bar = 0;
```

> **Tip:** To use **Prettify** instead of **Highlight.js**, just configure the **Markdown Extra** extension in the <i class="icon-cog"></i> **Settings** dialog.

> **Note:** You can find more information:

> - about **Prettify** syntax highlighting [here][5],
> - about **Highlight.js** syntax highlighting [here][6].


### Footnotes

You can create footnotes like this[^footnote].
 [^MVP]:[MVP](https://en.wikipedia.org/wiki/Model–view–presenter/) MVP모델에 대한 정의는 이 곳에서 확인할 수 있다.

### SmartyPants

SmartyPants converts ASCII punctuation characters into "smart" typographic punctuation HTML entities. For example:

|                  | ASCII                        | HTML              |
 ----------------- | ---------------------------- | ------------------
| Single backticks | `'Isn't this fun?'`            | 'Isn't this fun?' |
| Quotes           | `"Isn't this fun?"`            | "Isn't this fun?" |
| Dashes           | `-- is en-dash, --- is em-dash` | -- is en-dash, --- is em-dash |


### Table of contents

You can insert a table of contents using the marker `[TOC]`:

[TOC]


### MathJax

You can render *LaTeX* mathematical expressions using **MathJax**, as on [math.stackexchange.com][1]:

The *Gamma function* satisfying $\Gamma(n) = (n-1)!\quad\forall n\in\mathbb N$ is via the Euler integral

$$
\Gamma(z) = \int_0^\infty t^{z-1}e^{-t}dt\,.
$$

> **Tip:** To make sure mathematical expressions are rendered properly on your website, include **MathJax** into your template:

```
<script type="text/javascript" src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS_HTML"></script>
```

> **Note:** You can find more information about **LaTeX** mathematical expressions [here][4].


### UML diagrams

You can also render sequence diagrams like this:

```sequence
Alice->Bob: Hello Bob, how are you?
Note right of Bob: Bob thinks
Bob-->Alice: I am good thanks!
```

And flow charts like this:

```flow
st=>start: Start
e=>end
op=>operation: My Operation
cond=>condition: Yes or No?

st->op->cond
cond(yes)->e
cond(no)->op
```

> **Note:** You can find more information:

> - about **Sequence diagrams** syntax [here][7],
> - about **Flow charts** syntax [here][8].


 [^designPattern]: [design Pattern](https://stackedit.io/)  소프트웨어 개발 방법에서 사용되는 디자인 패턴은, 프로그램 개발에서 자주 나타나는 과제를 해결하기 위한 방법 중 하나로, 과거의 소프트웨어 개발 과정에서 발견된 설계의 노하우를 축적하여 이름을 붙여, 이후에 재이용하기 좋은 형태로 특정의 규약을 묶어서 정리한 것이다. - 위키피디아.

  [1]: http://math.stackexchange.com/
  [2]: http://daringfireball.net/projects/markdown/syntax "Markdown"
  [3]: https://github.com/jmcmanus/pagedown-extra "Pagedown Extra"
  [4]: http://meta.math.stackexchange.com/questions/5020/mathjax-basic-tutorial-and-quick-reference
  [5]: https://code.google.com/p/google-code-prettify/
  [6]: http://highlightjs.org/
  [7]: http://bramp.github.io/js-sequence-diagrams/
  [8]: http://adrai.github.io/flowchart.js/
