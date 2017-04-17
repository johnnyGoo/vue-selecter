<template>
    <div @touchstart="_onTouchStart"
         @mousedown="_onTouchStart" :style="style" :class="class">
        <slot></slot>
        <!--<p style="width: 1px;height: 1px;left: 50%;position: absolute;top:50%;background-color: #f00;"></p>-->
    </div>
</template>


/*!
* vue-bar v0.0.1 (https://github.com/johnnyGoo/vue-bar)
* Author: Johnny chen
*
* Copyright 2013-2016 Johnny chen
*/
<script>


    var count = 0;

    if (!window.Smart) {
        throw 'VueSelector required smart.js'
    }
    var Smart = window.Smart;
    var Css = Smart.Css;
    var _ = Smart._;
    var Utils = Smart.Utils;
    var tween;

    // 注册
    export default {
        // 声明 props
        props: {
            onlyExternal: {
                type: Boolean,
                default: false
            },
            active: {
                type: Number,
                default: 0
            },
            nodeSize: {
                type: Object,
                default: function () {
                    return {width: 100, height: 200, spacing: 50}
                }
            },
            scale: {
                type: Number,
                default: 1
            },
            time: {
                type: Number,
                default: 0.5
            },
            type: {
                type: String,
                default: 'x'
            }, midEffect: {
                type: Object,
                default: function () {
                    return {scale: 1.25, distance: 100}
                }
            }
            , style: {type: String, default: ''}
            , class: {type: String, default: ''}
            , isDraged: {
                type: Boolean,
                default: false
            }
        },
        data: function () {
            return {
                startPos: {pageX: 0, pageY: 0},
                startValue: {},
                display: false,
                dragging: false,
                children: null,
                info: null, value: {x: 0, y: 0},
                vector: null
            }

        },
        watch: {
            'value.x+value.y': function () {
                this._matchToBound(this.value);
            }
            , active: function (nv, ov) {
                this._showActive();
                this.$emit('update', nv);
            }
        }
        ,
        methods: {
            _showActive: function () {
                var me = this;
                var target = {};
                if (me.type == 'x') {
                    var midx = this.info.width / 2 - me.nodeSize.width / 2
                    var tox = -me.active * (me.nodeSize.width + me.nodeSize.spacing) + midx
                    target.x = tox;
                    //  this.value.x = tox;
                } else {
                    var midy = this.info.height / 2 - me.nodeSize.height / 2
                    var toy = -me.active * (me.nodeSize.height + me.nodeSize.spacing) + midy
                    target.y = toy;
                    //   this.value.y = toy;
                }

                if (tween) {
                    tween.kill();
                }


                tween = Smart.Tween.to(this.value, me.time, target)
                //                        .wait(0.5)
                        .ease(Smart.Tween.Sine.inOut)
//                        .start(function(){
//                            console.log('started');
//                        })
//                        .step(function(tween){
//                            console.log('step', tween.progress);
//                            console.log('value', tween.target)
//                        })
//                        .done(function(){
//                            console.log('done');
//                        })

            }
            ,
            _matchToBound(v){
                var me = this;
                var midx = this.info.width / 2
                var midy = this.info.height / 2
                var min_x = this.info.width / 2 - me.nodeSize.width / 2
                var max_x = min_x - (this.children.length * (me.nodeSize.width + me.nodeSize.spacing) - me.nodeSize.spacing)
                var min_y = this.info.height / 2 - me.nodeSize.height / 2
                var max_y = min_y - (this.children.length * (me.nodeSize.height + me.nodeSize.spacing) - me.nodeSize.spacing) + (me.nodeSize.height)

                if (me.type == 'x') {
                    if (v.x > min_x + me.midEffect.distance) {
                        v.x = min_x + me.midEffect.distance;
                    } else if (v.x < max_x - me.midEffect.distance) {
                        v.x = max_x - me.midEffect.distance;
                    }
                } else {
                    if (v.y > min_y + me.midEffect.distance) {
                        v.y = min_y + me.midEffect.distance;
                    } else if (v.y < max_y - me.midEffect.distance) {
                        v.y = max_y - me.midEffect.distance;
                    }
                }


                _.each(this.children, function (node, i) {
                    var x, y, reduce, scale, percent, cssObj, perspectiveCss, zindex;

                    if (me.type == 'x') {
                        y = (me.info.height - me.nodeSize.height) / 2;
                        x = (me.nodeSize.width + me.nodeSize.spacing) * i + v.x;
                        reduce = midx - me.nodeSize.width / 2 - x
                        percent = (1 - Math.min(Math.abs(reduce), me.midEffect.distance) / me.midEffect.distance)
                    } else {
                        x = (me.info.width - me.nodeSize.width) / 2;
                        y = (me.nodeSize.height + me.nodeSize.spacing) * i + v.y;
                        reduce = midy - me.nodeSize.height / 2 - y
                        percent = (1 - Math.min(Math.abs(reduce), me.midEffect.distance) / me.midEffect.distance)
                    }
                    scale = percent * (me.midEffect.scale - 1) + 1;
                    zindex = (percent * (me.midEffect['z-index'] )).toFixed(0);
                    var mx = 0, my = 0;

                    if (me.midEffect.x) {
                        mx = me.midEffect.x * percent
                    }
                    if (me.midEffect.y) {
                        my = me.midEffect.y * percent
                    }
                    if (percent <= 0) {
                        zindex = -Math.abs(i - me.active);
                    }


                    cssObj = {x: x + mx, y: y + my, scale: scale, 'z-index': zindex};
                    perspectiveCss = {};
                    reduce = reduce > 0 ? 1 : -1;

                    if (me.midEffect.rotateX) {
                        cssObj.rotateX = reduce * me.midEffect.rotateX * (1 - percent);
                    }
                    if (me.midEffect.rotateY) {
                        cssObj.rotateY = reduce * me.midEffect.rotateY * (1 - percent);
                    }
                    if (me.midEffect.perspective) {
                        perspectiveCss.perspective = me.midEffect.perspective
                    }
                    Css.smartCss(me.$el, perspectiveCss);

                    Css.smartCss(node, cssObj);
                });


                // Css.smartCss(this.$el.firstElementChild, {x: this.value.x, y: this.value.y})
            },
            _autoLock: function () {

                var me = this;
                var showID;
                if (me.type == 'x') {
                    var midx = this.info.width / 2 - me.nodeSize.width / 2
                    var maxx = midx - (this.children.length * (me.nodeSize.width + me.nodeSize.spacing) - me.nodeSize.spacing) + (me.nodeSize.width)


                    if (me.value.x > midx) {
                        showID = 0
                    } else if (me.value.x < maxx) {
                        showID = me.children.length - 1
                    } else {
                        showID = Math.floor((me.value.x - midx - me.nodeSize.width / 2 - me.nodeSize.spacing / 2) / (maxx - midx) * (this.children.length - 1));
                    }
//                    console.log(midx+':'+maxx+'    --'+showID)
                    me.active = showID
                    me._showActive();


                } else {
                    var midy = this.info.height / 2 - me.nodeSize.height / 2
                    var maxy = midy - (this.children.length * (me.nodeSize.height + me.nodeSize.spacing) - me.nodeSize.spacing) + (me.nodeSize.height)


                    if (me.value.y > midy) {
                        showID = 0
                    } else if (me.value.y < maxy) {
                        showID = this.children.length - 1
                    } else {
                        showID = Math.floor((me.value.y - midy - me.nodeSize.height / 2 - me.nodeSize.spacing / 2) / (maxy - midy) * (this.children.length - 1));

                    }
                    me.active = showID
                    me._showActive();
                }


            },
            _getTouchPos(e) {
                var tp = e.changedTouches ? e.changedTouches[0] : e;
                return {pageX: tp.pageX, pageY: tp.pageY};
            },
            __updateTime(t){
                _.each(this.children, function (node, i) {
                    Css.smartCss(node, {transition: 'all ' + t + 's ease'});
                })
            },
            _onTouchStart(e) {
                if (this.onlyExternal) {
                    return;
                }
                //    console.log('start '+e.type)

                //    this.__updateTime(0)
                this.startPos = this._getTouchPos(e);
                this.startValue = _.extend({}, this.value);
                this.dragging = true;
                this.isDraged = false;
                this.vector = new Smart.Physics.Vector();
                document.addEventListener('touchmove', this._onTouchMove, false);
                document.addEventListener('touchend', this._onTouchEnd, false);
                document.addEventListener('mousemove', this._onTouchMove, false);
                document.addEventListener('mouseup', this._onTouchEnd, false);
                // this.$emit('start', this.value);
            },
            _onTouchMove(e) {
                var npos = this._getTouchPos(e);
                this.value.x = (npos.pageX - this.startPos.pageX) / this.scale + this.startValue.x;
                this.value.y = (npos.pageY - this.startPos.pageY) / this.scale + this.startValue.y;
                this._matchToBound(this.value);
                if (this.isDraged == false) {
                    this.vector.point = new Smart.Physics.Point(npos.pageX - this.startPos.pageX, npos.pageY - this.startPos.pageY);
                    if (this.vector.length > 15) {
                        this.isDraged = true;
                    }
                }

                //  this.$emit('update', this.value);
            },
            _onTouchEnd(e) {
                //   this.__updateTime(this.time)
//                console.log('end '+e.type)
                this.dragging = false;
                document.removeEventListener('touchmove', this._onTouchMove);
                document.removeEventListener('touchend', this._onTouchEnd);
                document.removeEventListener('mousemove', this._onTouchMove);
                document.removeEventListener('mouseup', this._onTouchEnd);
                this._autoLock();
                //  this.$emit('end', this.value);

            }
        },
        computed: {},
        ready: function () {
            var me = this;

            if (!me.midEffect.distance) {
                me.midEffect.distance = 100
            }
            if (!me.midEffect.scale) {
                me.midEffect.scale = 1
            }


            this.info = Utils.domInfo(this.$el);
            this.children = []
            _.each(this.$el.childNodes, function (node) {
                if (_.contains(['DIV', 'IMG'], node.nodeName.toUpperCase())) {
                    me.children.push(node)
                    Css.smartCss(node, {transition: 'all ' + 0 + 's ease', position: 'absolute'});
                }
            });

            Css.smartCss(this.$el, {
                position: 'relative'//,
//                overflow: 'hidden'
            });
            //  this._matchToBound(this.value);
            me._showActive();


        }


    }


</script>