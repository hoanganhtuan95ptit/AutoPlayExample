## Dạo đầu
Xin chào anh em.
Bài viết này nói về cách mình đã dựng lên thư viện [DraggablePanel](https://github.com/hoanganhtuan95ptit/DraggablePanel) một thư viện xây dựng layout giống YOUTUBE. Thư viện được mình xây dựng trên những hiểu biết hiện có của mình nên sẽ có nhiều chỗ chưa thực sự tối ưu. Mình rất mong nhận được gạnh đá của anh em =))).
Giờ thì bắt đầu thôi nào!!

## XMl  [Link](https://github.com/hoanganhtuan95ptit/DraggablePanel/blob/master/draggable/src/main/res/layout/layout_draggable_panel.xml)
```java
<?xml version="1.0" encoding="utf-8"?>
<merge xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto">

    <com.hoanganhtuan95ptit.draggable.widget.DragFrame
        android:id="@+id/frameDrag"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:layout_alignParentBottom="true"
        android:layout_centerHorizontal="true"
        android:background="@android:color/transparent">

        <com.google.android.material.appbar.AppBarLayout
            android:id="@+id/appbarLayout"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:background="@android:color/transparent"
            app:elevation="0dp">

            <com.google.android.material.appbar.CollapsingToolbarLayout
                android:id="@+id/collapsingToolbarLayout"
                android:layout_width="match_parent"
                android:layout_height="match_parent"
                app:layout_scrollFlags="scroll|exitUntilCollapsed">

                <androidx.appcompat.widget.Toolbar
                    android:id="@+id/toolbar"
                    android:layout_width="match_parent"
                    android:layout_height="match_parent"
                    android:background="@android:color/transparent"
                    app:layout_collapseMode="pin" />
            </com.google.android.material.appbar.CollapsingToolbarLayout>
        </com.google.android.material.appbar.AppBarLayout>

        <FrameLayout
            android:id="@+id/frameSecond"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            app:layout_behavior="@string/appbar_scrolling_view_behavior" />

        <FrameLayout
            android:id="@+id/frameFirst"
            android:layout_width="match_parent"
            android:layout_height="match_parent" />

    </com.hoanganhtuan95ptit.draggable.widget.DragFrame>

</merge>
```

Nhìn đến đây chắc anh em cùng giống mình đang chửi thế thằng viết code. (DM thằng code, layout rắc rối vl) (DragFrame lá cái méo gì thế nhỉ??) (sao phải dùng CollapsingToolbarLayout, AppBarLayout, Toolbar). Nhưng từ từ đã anh em cái gì nó cũng có lý do của nó =))). Mình xin trả lời từ thắc mắc của các bạn.

Trước mình cũng chỉ dùng đến hai FrameLayout là frameFirst và frameSecond, nhưng rồi một ngày mình được xem video này [Cấm xem](https://www.youtube.com/watch?v=9RAqdgGXIj0&feature=share) bất ngờ chưa, nó là một video dọc. Là một fan chân chính với nhưng thế loại video nhẹ nhàng, tình cảm như: Anh thợ sửa ống nước may nắm, Cô hàng xóm... à nhầm  Xe đạp, Chờ anh trong con mưa, Quê tôi. Mình mới có cơ hội được xem những video doc và nhận ra rằng với các video đó youtube đã sử dụng cơ chế giống layout_behavior của CoordinatorLayout để có thể tối ưu nội dung bên dưới video (thực ra để nó tối ưu quảng cáo). 

(DragFrame lá cái méo gì thế nhỉ??) (sao phải dùng CollapsingToolbarLayout, AppBarLayout, Toolbar): nếu đọc nội dung bên trên thì chắc anh em cũng có câu trả cho mình rồi đúng không, vâng dragFrame là CoordinatorLayout và  CollapsingToolbarLayout, AppBarLayout, Toolbar là để hỗ trợ cho video dọc =))

Layout dựng xong, giờ là lúc phải xử lý chúng =)))

## CODE KOTLIN [Link](https://github.com/hoanganhtuan95ptit/DraggablePanel/blob/master/draggable/src/main/java/com/hoanganhtuan95ptit/draggable/DraggablePanel.kt)
 
 Đầu tiên Mình sẽ cần phải xây dựng cơ chế drag view 
```java
        private var downY = 0f
        private var deltaY = 0

        private var firstViewDown = false

        override fun onInterceptTouchEvent(ev: MotionEvent?): Boolean {
            when (ev!!.action) {
                MotionEvent.ACTION_DOWN -> {
                    firstViewDown = isViewUnder(frameFirst, ev)
                    downY = ev.rawY
                }
                MotionEvent.ACTION_MOVE -> {
                    checkFrameFirstMove(ev)
                }
            }
            return firstViewMove
        }

        override fun onTouchEvent(ev: MotionEvent?): Boolean {
            val motionY = ev!!.rawY.toInt()
            when (ev.action) {
                MotionEvent.ACTION_UP, MotionEvent.ACTION_CANCEL -> {
                    firstViewMove = false
                    firstViewDown = false
                    handleUp()
                }
                MotionEvent.ACTION_MOVE -> {
                    checkFrameFirstMove(ev)
                    if (firstViewMove) {
                        handleMove(motionY)
                    }
                }
            }
            return firstViewDown
        }
```

Khi nhắc đến drag thì có hai vấn đề chúng ta cần quan tâm đầu tiên là: chạm vào đâu (ACTION_DOWN) và khi nào di chuyển (ACTION_MOVE). Ở đây mình thiết lập bộ lắng nghe chạm trên frameDrag.

ACTION_DOWN: 
```java

        override fun onInterceptTouchEvent(ev: MotionEvent?): Boolean {
            when (ev!!.action) {
                MotionEvent.ACTION_DOWN -> {
                    firstViewDown = isViewUnder(frameFirst, ev)
                    downY = ev.rawY
                }
                MotionEvent.ACTION_MOVE -> {
                    checkFrameFirstMove(ev)
                }
            }
            return firstViewMove
        }
            ....
        private fun isViewUnder(view: View?, ev: MotionEvent): Boolean {
            return if (view == null) {
                false
            } else {
                ev.x >= view.left && ev.x < view.right && ev.y >= view.top && ev.y < view.bottom
            }
        }
```
isViewUnder Giúp mình phát hiện khi người dùng chạm vào frameDrag người dùng có đang chạm vào frameFirst không

ACTION_MOVE

```java

        override fun onInterceptTouchEvent(ev: MotionEvent?): Boolean {
            when (ev!!.action) {
                MotionEvent.ACTION_DOWN -> {
                    firstViewDown = isViewUnder(frameFirst, ev)
                    downY = ev.rawY
                }
                MotionEvent.ACTION_MOVE -> {
                    checkFrameFirstMove(ev)
                }
            }
            return firstViewMove
        }
            ....
        private fun checkFrameFirstMove(ev: MotionEvent) {
            if (firstViewMove) return
            val calculateDiff: Int = calculateDistance(ev, downY)
            val scaledTouchSlop: Int = getScaledTouchSlop(getContext())
            if (calculateDiff > scaledTouchSlop && firstViewDown) {
                deltaY = ev.rawY.toInt() - (frameDrag.layoutParams as LayoutParams).topMargin
                firstViewMove = true
            }
        }
```

checkFrameFirstMove giúp mình kiểm tra xem người dùng có đang di chuyển frameFirst hay không.

Ok vậy là chúng ta đã dựng xong bộ drag cho view, giờ là lúc chúng ta phải xử lý giao diện khi người dùng di chuyển frameFirst
```java
        private fun handleMove(motionY: Int) {
            setMarginTop(motionY - deltaY)
        }


        private fun setMarginTop(top: Int) {
            val marginTop = when {
                top < 0 -> 0
                top > mMarginTopWhenMin -> mMarginTopWhenMin
                else -> top
            }
    
            if (marginTop == mCurrentMarginTop) return
            mCurrentMarginTop = marginTop
    
            val percent: Float = mCurrentMarginTop * 1f / mMarginTopWhenMin
            setPercent(percent)
        }

        private fun setPercent(percent: Float) {
            if (mCurrentPercent == percent || percent > 1 || percent < 0) return
            mCurrentPercent = percent
    
            refresh()
        }

        /**
         * update ui all view
         */
        open fun refresh() {
    
            updateState()
    
            mDraggableListener?.onChangePercent(mCurrentPercent)
    
            val layoutParams = frameDrag.layoutParams as LayoutParams
            layoutParams.topMargin = (mMarginTopWhenMin * mCurrentPercent).toInt()
            layoutParams.leftMargin = (mMarginEdgeWhenMin * mCurrentPercent).toInt()
            layoutParams.rightMargin = (mMarginEdgeWhenMin * mCurrentPercent).toInt()
            layoutParams.bottomMargin = (mMarginBottomWhenMin * mCurrentPercent).toInt()
            frameDrag.layoutParams = layoutParams
    
            val toolBarHeight = when {
                firstViewMove -> {
                    (mHeightWhenMaxDefault - (mHeightWhenMaxDefault - mHeightWhenMinDefault) * mCurrentPercent).toInt()
                }
                mCurrentPercent == 0f -> {
                    mHeightWhenMaxDefault
                }
                else -> {
                    mHeightWhenMinDefault
                }
            }
    
            toolbar.reHeight(toolBarHeight)
    
            refreshFrameFirst()
        }
    
        /**
         * update ui frame first
         */
        open fun refreshFrameFirst() {
            val frameFistHeight = if (mCurrentPercent < mPercentWhenMiddle) {
                (mHeightWhenMax - (mHeightWhenMax - mHeightWhenMiddle) * mCurrentPercent / mPercentWhenMiddle)
            } else {
                (mHeightWhenMiddle - (mHeightWhenMiddle - mHeightWhenMin) * (mCurrentPercent - mPercentWhenMiddle) / (1 - mPercentWhenMiddle))
            }
    
            appbarLayout.reHeight(frameFistHeight.toInt())
        }                   
```

Khi người dùng di chuyển mình sẽ tính toán ra % dựa vào marginTop, Từ phần % hiện có mình sẽ tính toán để cập nhật kính thước cá view thành phần ở refresh (Cập nhật cho fragDrag và toolbar), refreshFrameFirst (cập nhật cho appbarLayout)
# Dillinger

[![N|Solid](https://cldup.com/dTxpPi9lDf.thumb.png)](https://nodesource.com/products/nsolid)

[![Build Status](https://travis-ci.org/joemccann/dillinger.svg?branch=master)](https://travis-ci.org/joemccann/dillinger)

Dillinger is a cloud-enabled, mobile-ready, offline-storage, AngularJS powered HTML5 Markdown editor.

  - Type some Markdown on the left
  - See HTML in the right
  - Magic

# New Features!

  - Import a HTML file and watch it magically convert to Markdown
  - Drag and drop images (requires your Dropbox account be linked)


You can also:
  - Import and save files from GitHub, Dropbox, Google Drive and One Drive
  - Drag and drop markdown and HTML files into Dillinger
  - Export documents as Markdown, HTML and PDF

Markdown is a lightweight markup language based on the formatting conventions that people naturally use in email.  As [John Gruber] writes on the [Markdown site][df1]

> The overriding design goal for Markdown's
> formatting syntax is to make it as readable
> as possible. The idea is that a
> Markdown-formatted document should be
> publishable as-is, as plain text, without
> looking like it's been marked up with tags
> or formatting instructions.

This text you see here is *actually* written in Markdown! To get a feel for Markdown's syntax, type some text into the left window and watch the results in the right.

### Tech

Dillinger uses a number of open source projects to work properly:

* [AngularJS] - HTML enhanced for web apps!
* [Ace Editor] - awesome web-based text editor
* [markdown-it] - Markdown parser done right. Fast and easy to extend.
* [Twitter Bootstrap] - great UI boilerplate for modern web apps
* [node.js] - evented I/O for the backend
* [Express] - fast node.js network app framework [@tjholowaychuk]
* [Gulp] - the streaming build system
* [Breakdance](https://breakdance.github.io/breakdance/) - HTML to Markdown converter
* [jQuery] - duh

And of course Dillinger itself is open source with a [public repository][dill]
 on GitHub.

### Installation

Dillinger requires [Node.js](https://nodejs.org/) v4+ to run.

Install the dependencies and devDependencies and start the server.

```sh
$ cd dillinger
$ npm install -d
$ node app
```

For production environments...

```sh
$ npm install --production
$ NODE_ENV=production node app
```

### Plugins

Dillinger is currently extended with the following plugins. Instructions on how to use them in your own application are linked below.

| Plugin | README |
| ------ | ------ |
| Dropbox | [plugins/dropbox/README.md][PlDb] |
| GitHub | [plugins/github/README.md][PlGh] |
| Google Drive | [plugins/googledrive/README.md][PlGd] |
| OneDrive | [plugins/onedrive/README.md][PlOd] |
| Medium | [plugins/medium/README.md][PlMe] |
| Google Analytics | [plugins/googleanalytics/README.md][PlGa] |


### Development

Want to contribute? Great!

Dillinger uses Gulp + Webpack for fast developing.
Make a change in your file and instantaneously see your updates!

Open your favorite Terminal and run these commands.

First Tab:
```sh
$ node app
```

Second Tab:
```sh
$ gulp watch
```

(optional) Third:
```sh
$ karma test
```
#### Building for source
For production release:
```sh
$ gulp build --prod
```
Generating pre-built zip archives for distribution:
```sh
$ gulp build dist --prod
```
### Docker
Dillinger is very easy to install and deploy in a Docker container.

By default, the Docker will expose port 8080, so change this within the Dockerfile if necessary. When ready, simply use the Dockerfile to build the image.

```sh
cd dillinger
docker build -t joemccann/dillinger:${package.json.version} .
```
This will create the dillinger image and pull in the necessary dependencies. Be sure to swap out `${package.json.version}` with the actual version of Dillinger.

Once done, run the Docker image and map the port to whatever you wish on your host. In this example, we simply map port 8000 of the host to port 8080 of the Docker (or whatever port was exposed in the Dockerfile):

```sh
docker run -d -p 8000:8080 --restart="always" <youruser>/dillinger:${package.json.version}
```

Verify the deployment by navigating to your server address in your preferred browser.

```sh
127.0.0.1:8000
```

#### Kubernetes + Google Cloud

See [KUBERNETES.md](https://github.com/joemccann/dillinger/blob/master/KUBERNETES.md)


### Todos

 - Write MORE Tests
 - Add Night Mode

License
----

MIT


**Free Software, Hell Yeah!**

[//]: # (These are reference links used in the body of this note and get stripped out when the markdown processor does its job. There is no need to format nicely because it shouldn't be seen. Thanks SO - http://stackoverflow.com/questions/4823468/store-comments-in-markdown-syntax)


   [dill]: <https://github.com/joemccann/dillinger>
   [git-repo-url]: <https://github.com/joemccann/dillinger.git>
   [john gruber]: <http://daringfireball.net>
   [df1]: <http://daringfireball.net/projects/markdown/>
   [markdown-it]: <https://github.com/markdown-it/markdown-it>
   [Ace Editor]: <http://ace.ajax.org>
   [node.js]: <http://nodejs.org>
   [Twitter Bootstrap]: <http://twitter.github.com/bootstrap/>
   [jQuery]: <http://jquery.com>
   [@tjholowaychuk]: <http://twitter.com/tjholowaychuk>
   [express]: <http://expressjs.com>
   [AngularJS]: <http://angularjs.org>
   [Gulp]: <http://gulpjs.com>

   [PlDb]: <https://github.com/joemccann/dillinger/tree/master/plugins/dropbox/README.md>
   [PlGh]: <https://github.com/joemccann/dillinger/tree/master/plugins/github/README.md>
   [PlGd]: <https://github.com/joemccann/dillinger/tree/master/plugins/googledrive/README.md>
   [PlOd]: <https://github.com/joemccann/dillinger/tree/master/plugins/onedrive/README.md>
   [PlMe]: <https://github.com/joemccann/dillinger/tree/master/plugins/medium/README.md>
   [PlGa]: <https://github.com/RahulHP/dillinger/blob/master/plugins/googleanalytics/README.md>
