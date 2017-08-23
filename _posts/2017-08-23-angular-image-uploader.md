---
layout: post
title:  "Angular 图片裁剪上传插件"
date:   2017-08-23 18:51:20
categories: front-end
---
本文将介绍基于Angular的图片裁剪上传插件。
github: https://github.com/licaomeng/angular-image-upload
插件效果如下：
![](https://rawgit.com/licaomeng/licaomeng.github.io/master/images/angular-image-uploader/1.gif)

该插件的图片裁剪是通过图片的放大、缩小、拖动完成的。而不同于我们通常所见到的拖动剪裁范围，进行的图片剪裁。这是一种反向思维。

**imgZoomCanvas.js**
----------------

图片的放大、缩小、拖动，全部是在html5的Canvas上面完成的。实现该算法的核心代码封装在 **imgZoomCanvas.js** 里面。

```
/**
 * Created by Caomeng Li on 8/23/2016.
 */
angular.module('appModule')
    .factory('imgZoomCanvas', [function () {

        //singleton
        var INSTANCE = null;

        var getInstance = function (options) {
            return INSTANCE || ( INSTANCE = new CanvasZoom(options) );
        }

        var destroyInstance = function () {
            if (INSTANCE) {
                INSTANCE = null;
            }
        }

        var stopAnimate = function () {
            return INSTANCE ? INSTANCE.stopAnimate() : null;
        }

        var onZoom = function (zoom) {
            return INSTANCE ? INSTANCE.doZoom(zoom) : null;
        }

        var CanvasZoom = function (options) {
            if (!options || !options.canvas) {
                throw 'CanvasZoom constructor: missing arguments canvas';
            }
            if (!options.image) {
                throw 'CanvasZoom constructor: missing arguments image';
            }

            this.canvas = options.canvas;
            this.image = options.image;
            this.currentAnimationId = 0;
            this.canvas.width = this.canvas.clientWidth;
            this.canvas.height = this.canvas.clientHeight;
            this.context = this.canvas.getContext('2d');

            this.lastX = 0;
            this.lastY = 0;

            this.position = {
                x: 0,
                y: 0
            };

            this.initPosition = {
                x: 0,
                y: 0
            }

            this.scale = {
                x: 1,
                y: 1
            };

            this.initScale = {
                x: 1,
                y: 1
            };

            this.init = false;

            this.checkRequestAnimationFrame();
            this.currentAnimationId = requestAnimationFrame(this.animate.bind(this));

            this.setEventListeners();
        }

        CanvasZoom.prototype = {
            stopAnimate: function () {
                cancelAnimationFrame(this.currentAnimationId);
            },

            animate: function () {
                this.context.clearRect(0, 0, this.canvas.width, this.canvas.height);
                var imgWidth = this.image.width,
                    imgHeight = this.image.height;
                if (!this.init) {
                    if (imgWidth > imgHeight) {
                        this.scale.x = this.scale.y = this.canvas.height / imgHeight;
                    } else {
                        this.scale.x = this.scale.y = this.canvas.width / imgWidth;
                    }
                    this.initScale.x = this.scale.x;
                    this.initScale.y = this.scale.y;
                }
                var currentWidth = (this.image.width * this.scale.x);
                var currentHeight = (this.image.height * this.scale.y);

                if (!this.init) {
                    if (imgWidth > imgHeight) {
                        this.position.x = (currentWidth - this.canvas.width) / 2;
                        this.position.x = this.position.x > 0 ? -this.position.x : this.position.x;
                    } else {
                        this.position.y = (currentHeight - this.canvas.height) / 2;
                        this.position.y = this.position.y > 0 ? -this.position.y : this.position.y;
                    }
                    this.initPosition.x = this.position.x;
                    this.initPosition.y = this.position.y
                    this.init = true;
                }

                this.context.drawImage(this.image, this.position.x, this.position.y, currentWidth, currentHeight);
                this.currentAnimationId = requestAnimationFrame(this.animate.bind(this));
            },

            doZoom: function (zoom) {
                if (!zoom) return;

                //new scale
                var currentScale = this.scale.x;
                var newScale = this.scale.x + zoom * this.scale.x / 100;

                //some helpers
                var deltaScale = newScale - currentScale;
                var currentWidth = (this.image.width * this.scale.x);
                var currentHeight = (this.image.height * this.scale.y);
                var deltaWidth = this.image.width * deltaScale;
                var deltaHeight = this.image.height * deltaScale;

                //by default scale doesnt change position and only add/remove pixel to right and bottom
                //so we must move the image to the left to keep the image centered
                //ex: coefX and coefY = 0.5 when image is centered <=> move image to the left 0.5x pixels added to the right
                var canvasmiddleX = this.canvas.clientWidth / 2;
                var canvasmiddleY = this.canvas.clientHeight / 2;
                var xonmap = (-this.position.x) + canvasmiddleX;
                var yonmap = (-this.position.y) + canvasmiddleY;
                var coefX = -xonmap / (currentWidth);
                var coefY = -yonmap / (currentHeight);
                var newPosX = this.position.x + deltaWidth * coefX;
                var newPosY = this.position.y + deltaHeight * coefY;

                //edges cases
                var newWidth = currentWidth + deltaWidth;
                var newHeight = currentHeight + deltaHeight;

                if (newPosX > 0) {
                    newPosX = 0;
                }
                if (newPosX + newWidth < this.canvas.clientWidth) {
                    newPosX = this.canvas.clientWidth - newWidth;
                }

                if (newHeight < this.canvas.clientHeight) return;
                if (newPosY > 0) {
                    newPosY = 0;
                }
                if (newPosY + newHeight < this.canvas.clientHeight) {
                    newPosY = this.canvas.clientHeight - newHeight;
                }

                //finally affectations
                this.scale.x = newScale;
                this.scale.y = newScale;
                this.position.x = newPosX;
                this.position.y = newPosY;

                //edge cases
                if (this.scale.x < this.initScale.x) {
                    this.scale.x = this.initScale.x;
                    this.scale.y = this.initScale.x;
                    this.position.x = this.initPosition.x;
                    this.position.y = this.initPosition.y;
                }
            },

            doMove: function (relativeX, relativeY) {
                if (this.lastX && this.lastY) {

                    console.log('relativeX', relativeX);
                    console.log('relativeY', relativeY);

                    console.log('this.lastX', this.lastX);
                    console.log('this.lastY', this.lastY);

                    var deltaX = relativeX - this.lastX;
                    var deltaY = relativeY - this.lastY;
                    console.log('deltaX', deltaX);
                    console.log('deltaY', deltaY);

                    var currentWidth = (this.image.width * this.scale.x);
                    var currentHeight = (this.image.height * this.scale.y);

                    this.position.x += deltaX;
                    this.position.y += deltaY;
                    console.log('this.position.x', this.position.x);
                    console.log('this.position.y', this.position.y);

                    // edge cases
                    if (this.position.x >= 0) {
                        this.position.x = 0;
                    } else if (this.position.x < 0 && this.position.x + currentWidth < this.canvas.width) {
                        this.position.x = this.canvas.width - Math.round(currentWidth);
                    }

                    if (this.position.y >= 0) {
                        this.position.y = 0;
                    } else if (this.position.y < 0 && this.position.y + currentHeight < this.canvas.height) {
                        this.position.y = this.canvas.height - Math.round(currentHeight);
                    }
                }
                this.lastX = relativeX;
                this.lastY = relativeY;
            },

            setEventListeners: function () {
                this.canvas.addEventListener('mousedown', function (e) {
                    this.mdown = true;
                    this.lastX = 0;
                    this.lastY = 0;
                }.bind(this));

                this.canvas.addEventListener('mouseup', function (e) {
                    this.mdown = false;
                }.bind(this));

                this.canvas.addEventListener('mousemove', function (e) {
                    var relativeX = e.pageX - this.canvas.getBoundingClientRect().left;
                    var relativeY = e.pageY - this.canvas.getBoundingClientRect().top;

                    if (e.target == this.canvas && this.mdown) {
                        this.doMove(relativeX, relativeY);
                    }

                    if (relativeX <= 0 || relativeX >= this.canvas.clientWidth || relativeY <= 0 || relativeY >= this.canvas.clientHeight) {
                        this.mdown = false;
                    }
                }.bind(this));
            },

            checkRequestAnimationFrame: function () {
                var lastTime = 0;
                var vendors = ['ms', 'moz', 'webkit', 'o'];
                for (var x = 0; x < vendors.length && !window.requestAnimationFrame; ++x) {
                    window.requestAnimationFrame = window[vendors[x] + 'RequestAnimationFrame'];
                    window.cancelAnimationFrame = window[vendors[x] + 'CancelAnimationFrame']
                        || window[vendors[x] + 'CancelRequestAnimationFrame'];
                }

                if (!window.requestAnimationFrame) {
                    window.requestAnimationFrame = function (callback, element) {
                        var currTime = new Date().getTime();
                        var timeToCall = Math.max(0, 16 - (currTime - lastTime));
                        var id = window.setTimeout(function () {
                            callback(currTime + timeToCall);
                        }, timeToCall);
                        lastTime = currTime + timeToCall;
                        return id;
                    };
                }

                if (!window.cancelAnimationFrame) {
                    window.cancelAnimationFrame = function (id) {
                        clearTimeout(id);
                    };
                }
            }
        }
        return {
            getInstance: getInstance,
            destroyInstance: destroyInstance,
            stopAnimate: stopAnimate,
            onZoom: onZoom
        };
    }]);
```
**imgZoomCanvas**开放出了四个方法：
```
getInstance (生成单体实例)
destroyInstance (销毁单体实例)
stopAnimate (停止requestAnimationFrame产生的动画)
onZoom (图片缩放。传入缩放的参数，放大为正，缩小为负)
```

**imgUploader.js**
-------------

接下来介绍**imgUpload**这个directive，就是上面那个GIF看到的图片上传组件。逻辑代码封装在imgUploader.js里面你可以自己定制这四个方法：

```
deleteAvatar(移除当前图片，并且上传到服务器)
uploadAvatar(上传当前图片)
zoomOut
zoomIn
(它们负责图片缩放的步长，为一正一负，参数可以自己慢慢调整。)
```
另外该directive开放出一个回调方法**onUpload**，可以在你的业务controller里面实现相关图片上传logic。onUpload有三个参数：

```
 (image, isDelete, isHasAvatar)
```
第一个image是从Canvas上面导出的base64具体的实现在刚才介绍的deleteAvatar中：
`canvas.toDataURL('png')`

**imgUploader**对应的html：

```
<div id="img-uploader">
    <canvas class="avatar" id="myCanvas" width="150" height="150"></canvas>
    <div style="position: relative;left:156px;top:-1px">
        <div id="delete-avatar" ng-click="deleteAvatar()" class="delete-avatar">
            <img src="./image/delete.png">
        </div>
        <div class="edit-avatar">
            <img src="./image/edit.png">

            <div id="container" class="container">
                <input class="file-picker" id="file" type="file"/>
            </div>
        </div>
        <div id="upload-avatar" ng-show="fileSelected" ng-click="uploadAvatar()" class="upload-avatar">
            <img src="./image/upload.png">
        </div>
        <div id="zoom-out" ng-show="fileSelected" ng-click="zoomOut()" class="zoom-out">
            <img src="./image/zoom_out.png">
        </div>
        <div id="zoom-in" ng-show="fileSelected" ng-click="zoomIn()" class="zoom-in">
            <img src="./image/zoom_in.png">
        </div>
    </div>
</div>
```
最后就是我们的页面逻辑代码了，页面controller中只需要实现上面提到的回调方法**onUpload**即可：

```
$scope.upload = function (image, isDelete, isHasAvatar) {
      // Write your image upload logic here
 }
```
页面html只需要加入我们刚才的directive imageUploader:

```
<img-uploader on-upload="upload(image, isDelete, isHasAvatar)" image="image" is-editable="isEditable"></img-uploader>
```