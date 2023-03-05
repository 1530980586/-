// ==UserScript==
// @name         🥇超星/学习通网课小助手|修复视频播放|自动跳转任务点|章测作业字体解密|自动答题|题库覆盖率高、完全免费
// @namespace    noshuang
// @version      3.5.2
// @description  ✨超星学习通视频、章节测试、作业小助手，目前支持以下功能⚪视频自动观看，跳转下一个任务点⚪章节测试自动完成，无答案保存⚪作业自动完成，自动保存
// @author       noshuang
// @match        *://*.chaoxing.com/*
// @connect      cx.icodef.com
// @connect      s.jiaoyu139.com
// @run-at       document-end
// @grant        unsafeWindow
// @grant        GM_addStyle
// @grant        GM_getResourceText
// @grant        GM_xmlhttpRequest
// @grant        GM_setClipboard
// @require      https://lib.baomitu.com/jquery/2.0.0/jquery.min.js
// @require      https://cdn.jsdelivr.net/gh/photopea/Typr.js@15aa12ffa6cf39e8788562ea4af65b42317375fb/src/Typr.min.js
// @require      https://cdn.jsdelivr.net/gh/photopea/Typr.js@f4fcdeb8014edc75ab7296bd85ac9cde8cb30489/src/Typr.U.min.js
// @resource     Table https://www.forestpolice.org/ttf/2.0/table.json
// @license      MIT
// @original-script https://greasyfork.org/scripts/369625
// @original-author wyn665817
// @original-license MIT
// ==/UserScript==

// 设置修改后，需要刷新或重新打开网课页面才会生效(0关闭，1开启)
var setting = {
        // 5E3 == 5000，科学记数法，表示毫秒数
        time: 5E3 // 默认响应速度为5秒，不建议小于3秒
        , review: 0 // 复习模式，完整挂机视频(音频)时长，支持挂机任务点已完成的视频和音频
        , queue: 1 // 队列模式，开启后任务点逐一完成，关闭则单页面所有任务点同时进行

        // 1代表开启，0代表关闭
        , video: 1 // 视频支持后台、切换窗口不暂停，支持多视频
        , work: 1 // 自动答题功能(章节测验)，作业需要手动开启查询，高准确率
        , audio: 1 // 音频自动播放，与视频功能共享vol和rate参数
        , book: 1 // 图书阅读任务点，非课程阅读任务点
        , docs: 1 // 文档阅读任务点，PPT类任务点自动完成阅读任务
        , jump: 1 // 自动切换标签、章节、课程(需要配置course参数)
        , read: '10' // 挂机课程阅读时间，单位分钟
        , total: 1 // 显示课程进度的统计数据，在学习进度页面的上方展示

        // 仅开启video(audio)时，修改此处才会生效
        , line: '公网1' // 视频播放的默认资源线路
        , http: '标清' // 视频播放的默认清晰度
        , vol: '0' // 默认音量的百分数，设定范围：[0,100]，'0'为静音
        , rate: '0' // 倍速已经失效，建议一倍速看，多倍速会异常提示

        // 仅开启work时，修改此处才会生效
        , auto: 1 // 答题完成后自动提交，默认开启
        , none: 0 // 无匹配答案时执行默认操作，关闭后若题目无匹配答案则会暂时保存已作答的题目
        , scale: 1 // 富文本编辑器高度自动拉伸，用于文本类题目，答题框根据内容自动调整大小
        , gs: 1 // 焦点跟随

        // 仅开启jump时，修改此处才会生效
        , course: 1 // 当前课程完成后自动切换课程，仅支持按照根目录课程顺序切换，默认关闭
        , lock: 1 // 跳过未开放(图标是锁)的章节，即闯关模式或定时发放的任务点，默认开启

    },
    _self = unsafeWindow,
    url = location.pathname,
    top = _self,
    api = 'http://cx.icodef.com/wyn-nb?v=4';

// 页面中转
if (url != '/studyApp/studying' && top != _self.top) document.domain = location.host.replace(/.+?\./, '');
try {
    while (top != _self.top) {
        top = top.parent.document ? top.parent : _self.top;
        if (top.location.pathname == '/mycourse/studentstudy') break;
    }
} catch (err) {
    // console.log(err);
    top = _self;
}

var $ = _self.jQuery || top.jQuery,
    parent = _self == top ? self : _self.parent,
    Ext = _self.Ext || parent.Ext || {},
    UE = _self.UE,
    vjs = _self.videojs;

$('.Header').find('a:contains(回到旧版)')[0] ? $('.Header').find('a:contains(回到旧版)')[0].click() : '';

String.prototype.toCDB = function () {
    return this.replace(/\s/g, '').replace(/[\uff01-\uff5e]/g, function (str) {
        return String.fromCharCode(str.charCodeAt(0) - 65248);
    }).replace(/[“”]/g, '"').replace(/[‘’]/g, "'").replace(/。/g, '.');
};
_self.alert = console.log;

setting.normal = ''; // ':visible'
setting.job = [':not(*)'];
setting.video && setting.job.push('iframe[src*="/video/index.html"]');
setting.work && setting.job.push('iframe[src*="/work/index.html"]');
setting.audio && setting.job.push('iframe[src*="/audio/index.html"]');
setting.book && setting.job.push('iframe[src*="/innerbook/index.html"]');
setting.docs && setting.job.push('iframe[src*="/ppt/index.html"]', 'iframe[src*="/pdf/index.html"]');

if (url == '/mycourse/studentstudy') {
    showTips();
    _self.checkMobileBrowerLearn = $.noop;
    var classId = location.search.match(/cla[zs]{2}id=(\d+)/i)[1] || 0,
        courseId = _self.courseId || location.search.match(/courseId=(\d+)/i)[1] || 0;
    !setting.jump || setting.lock || $('#coursetree').on('click', '[onclick*=void], [href*=void]', function () {
        _self.getTeacherAjax(courseId, classId, $(this).parent().attr('id').slice(3));
    });
} else if (url == '/ananas/modules/video/index.html' && setting.video) {
    if (setting.review) _self.greenligth = Ext.emptyFn;
    passVideo()
} else if (url == '/work/doHomeWorkNew' || url == '/api/work' || url == '/work/addStudentWorkNewWeb') {
    top.courseId = location.search.match(/courseId=(\d+)/i)[1];
    if (!UE) {
        var len = ($ || Ext.query || Array)('font:contains(未登录)', document).length;
        setTimeout(len == 1 ? top.location.reload : parent.greenligth, setting.time);
    } else if (setting.work) {
        setTimeout(relieveLimit, 0);
        beforeFind();
    }
} else if (url == '/ananas/modules/innerbook/index.html' && setting.book) {
    setTimeout(function () {
        if (getTip()) _self.setting ? _self.top.onchangepage(_self.getFrameAttr('end')) : _self.greenligth();
    }, setting.time);
} else if (url.match(/^\/ananas\/modules\/(ppt|pdf)\/index\.html$/) && setting.docs) {
    setTimeout(function () {
        if (getTip()) _self.setting ? _self.finishJob() : _self.greenligth();
    }, setting.time);
    frameElement.setAttribute('download', 1);
    if (_self.downloadNum == '') _self.downloadNum = '1';
} else if (url == '/knowledge/cards') {
    $ && checkToNext();
} else if (url.match(/^\/(course|zt)\/\d+\.html$/)) {
    setTimeout(function () {
        +setting.read && _self.sendLogs && $('.course_section:eq(0) .chapterText').click();
    }, setting.time);
} else if (url == '/ztnodedetailcontroller/visitnodedetail') {
    setting.read *= 60 / $('.course_section').length;
    setting.read && _self.sendLogs && autoRead();
} else if (url == '/mycourse/studentcourse') {
    var gv = location.search.match(/d=\d+&/g);
    setting.total && $('<a>', {
        href: '/moocAnalysis/chapterStatisticByUser?classI' + gv[1] + 'courseI' + gv[0] + 'userId=' + _self.getCookie('_uid') + '&ut=s',
        target: '_blank',
        title: '点击查看章节统计',
        style: 'margin: 0 25px;',
        html: '本课程共' + $('.icon').length + '节，剩余' + $('em:not(.openlock)').length + '节未完成'
    }).appendTo('.zt_logo').parent().width('auto');
} else if (location.hostname == 'i.mooc.chaoxing.com') {
} else if (url == '/work/selectWorkQuestionYiPiYue') {
    submitAnswer(getIframe().parent(), $.extend(true, [], parent._data));
} else if (url == '/exam-ans/exam/test') {
    showTips()
    $('.page').eq(0).html("<a style='font-weight: bold'>本脚本因为已知的不可抗力原因无法使用考试功能噢，可以使用左上角图片的app尝试考试，<br/>正规考试请勿作弊！</a>");
}
;

function whatttf() {
    var $tip = $('style:contains(font-cxsecret)');
    if (!$tip.length) return;
    var font = $tip.text().match(/base64,([\w\W]+?)'/)[1];
    font = Typr.parse(base64ToUint8Array(font))[0];
    var table = JSON.parse(GM_getResourceText('Table'));
    var match = {};
    for (var i = 19968; i < 40870; i++) { // 中文[19968, 40869]
        $tip = Typr.U.codeToGlyph(font, i);
        if (!$tip) continue;
        $tip = Typr.U.glyphToPath(font, $tip);
        $tip = $.md5(JSON.stringify($tip)).slice(24); // 8位即可区分
        match[i] = table[$tip];
    }

    // 替换加密字体
    $('.font-cxsecret').html(function (index, html) {
        $.each(match, function (key, value) {
            key = String.fromCharCode(key);
            key = new RegExp(key, 'g');
            value = String.fromCharCode(value);
            html = html.replace(key, value);
        });
        return html;
    }).removeClass('font-cxsecret'); // 移除字体加密

    function base64ToUint8Array(base64) {
        var data = window.atob(base64);
        var buffer = new Uint8Array(data.length);
        for (var i = 0; i < data.length; ++i) {
            buffer[i] = data.charCodeAt(i);
        }
        return buffer;
    }
}

function time_to_sec(time) {
    let s = 0;
    let t = 1;
    for (let i = time.split(':').length - 1; i >= 0; i--) {
        s += Number(time.split(':')[i]) * t
        t *= 60
    }
    return s;
};

function getTip() {
    return top != _self && jobSort($ || Ext.query);
}

function passVideo() {
    // rateHack()
    getTip() && setTimeout(() => {
        let vd = $('video')[0];
        if (vd) {
            console.log('播放视频');
            vd.volume = vd.volume === 0 ? 1 : 0;
            $('.vjs-big-play-button')[0].click();
        }
    }, 2000)
}

function jobSort($) {
    var fn = $.fn ? [getIframe(1), 'length'] : [self, 'dom'],
        sel = setting.job.join(', :not(.ans-job-finished) > .ans-job-icon' + setting.normal + ' ~ ');
    if (!getIframe()[fn[1]] || getIframe().parent().is('.ans-job-finished')) return 0;
    if (!setting.queue || $(sel, fn[0].parent.document)[0] == fn[0].frameElement) return 1;
    setInterval(function () {
        $(sel, fn[0].parent.document)[0] == fn[0].frameElement && fn[0].location.reload();
    }, setting.time);
}

function getIframe(tip, win, job) {
    if (!$) return Ext.get(frameElement || []).parent().child('.ans-job-icon') || Ext.get([]);
    do {
        win = win ? win.parent : _self;
        job = $(win.frameElement).prevAll('.ans-job-icon');
    } while (!job.length && win.parent.frameElement);
    return tip ? win : job;
}

function relieveLimit() {
    if (setting.scale) _self.UEDITOR_CONFIG.scaleEnabled = false;
    $.each(UE.instants, function () {
        var key = this.key;
        this.ready(function () {
            this.destroy();
            UE.getEditor(key);
        });
    });
}

function showTips() {
    setting.div = _self.top.$(
        '<div class="article-list" style="font-family: Microsoft Yahei,serif;display: flex;flex-direction: column;margin: 10px; max-height:800px; width:350px; position: fixed; top: 0; left: 0; z-index: 99999; overflow-x: auto;">' +
        '<div class="has-image" style="display: flex;flex-direction: column;background-color: #fff;box-shadow: 0px 4px 8px rgba(0, 0, 0, 0.04), 0px 0px 2px rgba(0, 0, 0, 0.06), 0px 0px 1px rgba(0, 0, 0, 0.04);border-radius: 10px;overflow: hidden;transition: box-shadow .3s ease;">' +
        '<div class="article-header">' +
        '<div class="article-image">' +
        '<img style="height: 150px;" src="http://img1.16q.cn/86fdcef2bcd4ff1d879631420f7b8397">' +
        '</div>' +
        '<div class="article-title" style="background: #f5f5fa; display: flex;flex-direction: column;justify-content: center;padding: 10px;gap: 8px;">' +
        '<a>欢迎使用脚本，本脚本提供以下功能：<br/>1.视频（目前仅1倍速，多倍会退回，如果失效了请联系作者修复）<br/>2.章节测试<br/>3.作业（已解决字体加密）<a/>' +
        '<a style="font-weight: bold;">如果您有作业考试等任务，可以扫上面图片中的二维码下载app搜题。</a>' +
        '<a>视频将在页面加载5秒后播放视频......（失效请联系作者）</a>' +
        '</div>' +
        '</div>'
    ).appendTo('body')
    drag();
}

function beforeFind() {
    whatttf();
    setting.regl = parent.greenligth || $.noop;
    if ($.type(parent._data) == 'array') return setting.regl();
    _self.top.$('.article-list').remove();
    setting.div = _self.top.$(
        '<div class="article-list" style="font-family: Microsoft Yahei,serif;display: flex;flex-direction: column;margin: 10px; width:350px; position: fixed; top: 0; left: 0; z-index: 99999; overflow-x: auto;">' +
        '<div class="has-image" style="display: flex;flex-direction: column;background-color: #fff;box-shadow: 0px 4px 8px rgba(0, 0, 0, 0.04), 0px 0px 2px rgba(0, 0, 0, 0.06), 0px 0px 1px rgba(0, 0, 0, 0.04);border-radius: 10px;overflow: hidden;transition: box-shadow .3s ease;">' +
        '<div class="article-header">' +
        '<div class="article-image">' +
        '<img style="height: 150px;" src="http://img1.16q.cn/86fdcef2bcd4ff1d879631420f7b8397">' +
        '</div>' +
        '<div class="article-title" style="background: #f5f5fa; display: flex;flex-direction: column;justify-content: center;padding: 10px;gap: 8px;">' +
        '<a>欢迎使用脚本，本脚本提供以下功能：<br/>1.视频（目前仅1倍速，多倍会退回，如果失效了请联系作者修复）<br/>2.章节测试<br/>3.作业（已解决字体加密）<a/>' +
        '<a style="font-weight: bold;">如果有搜不到的可以尝试扫上面图片的二维码下载app搜题，也可以先下载备用。</a>' +
        '<a ">作业在搜索框加载后答题......（失效请联系作者）</a>' +
        '<div>' +
        '<button class="ui primary basic button"" style="margin-bottom: 10px;border:0;background-color: #2a9d8f;color: #fff;padding: 8px 16px;width: 100px;">暂停答题</button>' +
        '<button class="ui primary basic button"" style="margin-bottom: 10px;border:0;background-color: #2a9d8f;color: #fff;padding: 8px 16px;width: 150px;margin-left: 10px;">' + (setting.auto ? '取消本次自动提交' : '开启本次自动提交') + '</button>' +
        '<br/>' +
        '<button style="border:0;background-color: #2a9d8f;color: #fff;padding: 8px 16px;width: 100px;">重新查询</button>' +
        '<button style="border:0;background-color: #2a9d8f;color: #fff;padding: 8px 16px;width: 100px;margin-left: 15px;">折叠搜题框</button>' +
        '<button class="btn btn-primary" style="border:0;background-color: #2a9d8f;color: #fff;padding: 8px 16px;width: 100px;margin-left: 15px;">取消跟随</button>' +
        '</div>' +
        '<div style="overflow-y: auto;max-height: 300px;">' +
        '<table  class="layui-table" style="font-size: 12px;">' +
        '<thead>' +
        '<tr>' +
        '<th style="width: 30px; min-width: 30px; font-weight: bold; text-align: center;">题号</th>' +
        '<th style="width: 60%; min-width: 130px; font-weight: bold; text-align: center;">题目（点击可复制）</th>' +
        '<th style="min-width: 130px; font-weight: bold; text-align: center;">答案（点击可复制）</th>' +
        '</tr>' +
        '</thead>' +
        '<tfoot style="display: none;">' +
        '<tr>' +
        '<th colspan="3" style="font-weight: bold; text-align: center;">答案提示框 已折叠</th>' +
        '</tr>' +
        '</tfoot>' +
        '<tbody>' +
        '<tr>' +
        '<td colspan="3">扫二维码题库查询更全</td>' +
        '</tr>' +
        '</tbody>' +
        '</table>' +
        '</div>' +
        '</div>' +
        '</div>'
    ).appendTo('body').on('click', 'button, td', function () {
        var len = $(this).prevAll('button').length;
        if (this.nodeName == 'TD') {
            $(this).prev().length && GM_setClipboard($(this).text());
        } else if (!$(this).siblings().length) {
            $(this).parent().html('<h3>正在搜索答案...<h3>');
            setting.num++;
        } else if (len === 0) {
            if (setting.loop) {
                clearInterval(setting.loop);
                delete setting.loop;
                len = ['已暂停搜索', '继续答题'];
            } else {
                setting.loop = setInterval(findAnswer, setting.time);
                len = ['正在搜索答案...', '暂停答题'];
            }
            setting.div.children('div:eq(0)').html(function () {
                return $(this).data('html') || len[0];
            }).removeData('html');
            $(this).html(len[1]);
        } else if (len == 1) {
            setting.auto = !setting.auto;
            $(this).html(setting.auto ? '取消本次自动提交' : '开启本次自动提交');
        } else if (len == 2) {
            parent.location.reload();
        } else if (len == 3) {
            setting.div.find('tbody, tfoot').toggle();
        } else if (len == 4) {
            setting.gs = !setting.gs;
            $(this).html(setting.gs ? '取消跟随' : '开启跟随');
        }
    }).find('table, td, th').css('border', '1px solid').end();
    drag();
    setting.lose = setting.num = 0;
    setting.data = parent._data = [];
    setting.over = '<button style="margin-right: 10px;">跳过此题</button>';
    setting.curs = $('script:contains(courseName)', top.document).text().match(/courseName:\'(.+?)\'|$/)[1] || $('h1').text().trim() || '无';
    setting.loop = setInterval(findAnswer, setting.time);
    var tip = ({undefined: '任务点排队中', 0: '等待切换中'})[getTip()];
    tip && setting.div.children('div:eq(0)').data('html', tip).siblings('button:eq(0)').click();
}

function drag() {
    var $article = _self.top.$('.article-list')
    $article.on('mousedown', function (e) {
        var x = e.clientX,
            y = e.clientY,
            l = $article.position().left,
            t = $article.position().top;
        // 获取滚动条已经下滑的长度
        var scrollTop = _self.top.document.documentElement.scrollTop || _self.top.document.body.scrollTop;
        $(_self.top.document).on('mousemove', function (e) {
            console.log(e.clientX, e.clientY)
            $article.css({
                left: l + e.clientX - x,
                top: t + e.clientY - y - scrollTop
            });
        }).on('mouseup', function () {
            $(this).off('mousemove mouseup');
        });
    });
}

function findAnswer() {
    if (setting.num >= $('.TiMu').length) {
        var arr = setting.lose ? ['共有 <font color="red">' + setting.lose + '</font> 道题目待完善（已深色标注）', saveThis] : ['答题已完成', submitThis];
        setting.div.children('div:eq(0)').data('html', arr[0]).siblings('button:eq(0)').hide().click();
        return setTimeout(arr[1], setting.time);
    }
    var $TiMu = $('.TiMu').eq(setting.num);
    var question = filterImg($TiMu.find('.Zy_TItle:eq(0) .clearfix')).replace(/^【.*?】\s*/, '').replace(/\s*（\d+\.\d+分）$/, ''),
        type = $TiMu.find('input[name^=answertype]:eq(0)').val() || '-1';
    if (setting.gs) {
        $("html,body").animate({scrollTop: $('.TiMu').eq(setting.num).offset().top}, 1000);
    }

    GM_xmlhttpRequest({
        method: 'POST',
        url: api,
        headers: {
            'Content-type': 'application/x-www-form-urlencoded'
        },
        data: 'question=' + encodeURIComponent(question),
        timeout: setting.time,
        onload: function (xhr) {
            if (!setting.loop) {
            } else if (xhr.status == 200) {
                var obj = $.parseJSON(xhr.responseText) || {};
                obj.answer = obj.data;
                if (obj.code) {
                    // setting.div.children('div:eq(0)').html('<i class="layui-icon layui-icon-loading"></i>  正在搜索答案.....');
                    var td = '<td style="border: 1px solid;',
                        answer = obj.answer.replace(/&/g, '&amp;').replace(/<(?!img)/g, '&lt;');
                    obj.answer = /^http/.test(answer) ? '<img src="' + obj.answer + '">' : obj.answer;

                    $(
                        '<tr>' +
                        td + ' text-align: center;">' + $TiMu.find('.Zy_TItle:eq(0) i').text().trim() + '</td>' +
                        td + '" title="点击可复制">' + (question.match('<img') ? question : question.replace(/&/g, '&amp;').replace(/</g, '&lt')) + '</td>' +
                        td + '" title="点击可复制">' + (/^http/.test(answer) ? obj.answer : '') + obj.answer + '</td>' +
                        '</tr>'
                    ).appendTo(setting.div.find('tbody')).css('background-color', fillAnswer($TiMu.find('ul:eq(0)').find('li'), obj, type) ? '' : 'rgba(0, 150, 136, 0.6)');
                    setting.num++;
                } else {
                    setting.div.children('div:eq(0)').html('<h4>扫码题库更全<img src="https://cdn.fm210.cn/img/huoxin.png" style="width:100px;" class="img-thumbnail"><h4>');
                }
                setting.div.children('span').html('<h4>扫码题库更全<img src="https://cdn.fm210.cn/img/huoxin.png" style="width:100px;" class="img-thumbnail"><h4>');
            } else if (xhr.status == 403) {
                var html = xhr.responseText.indexOf('{') ? '<h4>扫码题库更全<img src="https://cdn.fm210.cn/img/huoxin.png" style="width:100px;" class="img-thumbnail"><h4>' : $.parseJSON(xhr.responseText).data;
                setting.div.children('div:eq(0)').data('html', html).siblings('button:eq(0)').click();
            } else {
                setting.div.children('div:eq(0)').html('<h4>扫码题库更全<img src="https://cdn.fm210.cn/img/huoxin.png" style="width:100px;" class="img-thumbnail"><h4>');
            }
        },
        ontimeout: function () {
            setting.loop && setting.div.children('div:eq(0)').html('<h4>扫码题库更全<img src="https://cdn.fm210.cn/img/huoxin.png" style="width:100px;" class="img-thumbnail"><h4>');
        }
    });
}

function fillAnswer($li, obj, type) {
    var $input = $li.find(':radio, :checkbox'),
        str = String(obj.data).toCDB() || new Date().toString(),
        data = str.split(/#|\x01|\|/),
        opt = obj.opt || str,
        state = setting.lose;
    // $li.find(':radio:checked').prop('checked', false);
    obj.code > 0 && $input.each(function (index) {
        if (this.value == 'true') {
            data.join().match(/(^|,)(正确|是|对|√|T|ri|right|true)(,|$)/) && this.click();
        } else if (this.value == 'false') {
            data.join().match(/(^|,)(错误|否|错|×|F|wr|wrong|false)(,|$)/) && this.click();
        } else {
            var tip = filterImg($li.eq(index).find('.after')).toCDB() || new Date().toString();
            Boolean($.inArray(tip, data) + 1 || (type == '1' && str.indexOf(tip) + 1)) == this.checked || this.click();
        }
    }).each(function () {
        if (!/^A?B?C?D?E?F?G?$/.test(opt)) return false;
        Boolean(opt.match(this.value)) == this.checked || this.click();
    });
    if (type.match(/^[013]$/)) {
        $input.is(':checked') || (setting.none ? ($input[Math.floor(Math.random() * $input.length)] || $()).click() : setting.lose++);
    } else if (type.match(/^(2|[4-9]|1[08])$/)) {
        data = String(obj.data).split(/#|\x01|\|/);
        str = $li.end().find('textarea').each(function (index) {
            index = (obj.code > 0 && data[index]) || this.value || '';
            UE.getEditor(this.name).setContent(index.trim());
        }).length;
        (obj.code == 1 && data.length == str) || setting.none || setting.lose++;
    } else {
        setting.none || setting.lose++;
    }
    return state == setting.lose;
}

function saveThis() {
    if (!setting.auto) return setTimeout(saveThis, setting.time);
    setting.div.children('button:lt(3)').hide().eq(1).click();
    $('#tempsave').click();
    setting.regl();
}

function submitThis() {
    if (!setting.auto) {
    } else if (!$('.Btn_blue_1:visible').length) {
        setting.div.children('button:lt(3)').hide().eq(1).click();
        return setting.regl();
    } else if ($('#confirmSubWin:visible').length) {
        var btn = $('#tipContent + * > a').offset() || {top: 0, left: 0},
            mouse = document.createEvent('MouseEvents');
        btn = [btn.left + Math.ceil(Math.random() * 46), btn.top + Math.ceil(Math.random() * 26)];
        mouse.initMouseEvent('click', true, true, document.defaultView, 0, 0, 0, btn[0], btn[1], false, false, false, false, 0, null);
        _self.event = $.extend(true, {}, mouse);
        delete _self.event.isTrusted;
        _self.form1submit();
    } else {
        $('.Btn_blue_1')[0].click();
    }
    setTimeout(submitThis, Math.ceil(setting.time * Math.random()) * 2);
}

function checkToNext() {
    var $tip = $(setting.job.join(', '), document).prevAll('.ans-job-icon' + setting.normal);
    setInterval(function () {
        $tip.parent(':not(.ans-job-finished)').length || setting.jump && toNext();
    }, setting.time);
}

function toNext() {
    let $ = _self.parent.$;
    var $cur = $('#cur' + $('#chapterIdid').val()),
        $tip = $('span.currents ~ span'),
        sel = setting.review ? 'html' : '.blue';
    if (!$cur.has(sel).length && $tip.length) return $tip.eq(0).click();
    $tip = $('.roundpointStudent, .roundpoint').parent();
    $tip = $tip.slice($tip.index($cur) + 1).not(':has(' + sel + ')');
    $tip.not(setting.lock ? ':has(.lock)' : 'html').find('span').eq(0).click();
    $tip.length || setting.course && switchCourse();
}

function switchCourse() {
    GM_xmlhttpRequest({
        method: 'GET',
        url: '/visit/courses/study?isAjax=true&fileId=0&debug=',
        headers: {
            'Referer': location.origin + '/visit/courses',
            'X-Requested-With': 'XMLHttpRequest'
        },
        onload: function (xhr) {
            var $list = $('.curFile .courseName', xhr.responseText),
                index = $list.index($list.filter('[href*="courseid=' + top.courseId + '&"]')) + 1;
            index = index && $list.eq(index).attr('href');
            setting.course = index && goCourse(index);
        }
    });
}

function goCourse(url) {
    GM_xmlhttpRequest({
        method: 'GET',
        url: url,
        onload: function (xhr) {
            $.globalEval('location.href = "' + $('.articlename a[href]', xhr.responseText).attr('href') + '";');
        }
    });
}

function autoRead() {
    $('html, body').animate({
        scrollTop: $(document).height() - $(window).height()
    }, Math.round(setting.read) * 1E3, function () {
        $('.nodeItem.r i').click();
    }).one('click', '#top', function (event) {
        $(event.delegateTarget).stop();
    });
}


function submitAnswer($job, data) {
    $job.removeClass('ans-job-finished');
    data = data.length ? $(data) : $('.TiMu').map(function () {
        var title = filterImg($('.Zy_TItle .clearfix', this));
        return {
            question: title.replace(/^【.*?】\s*/, ''),
            type: ({单选题: 0, 多选题: 1, 填空题: 2, 判断题: 3})[title.match(/^【(.*?)】|$/)[1]]
        };
    });
    data = $.grep(data.map(function (index) {
        var $TiMu = $('.TiMu').eq(index);
        if (!($.isPlainObject(this) && this.type < 4 && $TiMu.find('.fr').length)) {
            return false;
        } else if (this.type == 2) {
            var $ans = $TiMu.find('.Py_tk, .Py_answer').eq(0);
            if (!$TiMu.find('.cuo').length && this.code) {
                return false;
            } else if (!$ans.find('.cuo').length) {
                this.option = $ans.find('.clearfix').map(function () {
                    return filterImg(this);
                }).get().join('#') || '无';
            } else if (this.code) {
                this.code = -1;
            } else {
                return false;
            }
        } else if (this.type == 3) {
            var ans = $TiMu.find('.font20:last').text();
            if ($TiMu.find('.cuo').length) {
                this.option = ({'√': '错误', '×': '正确'})[ans] || '无';
            } else if (!this.code) {
                this.option = ({'√': '正确', '×': '错误'})[ans] || '无';
            } else {
                return false;
            }
        } else {
            var text = $TiMu.find('.Py_answer > span:eq(0)').text();
            if ($TiMu.find('.dui').length && this.code && !/^A?B?C?D?E?F?G?$/.test(this.option)) {
                return false;
            } else if ($TiMu.find('.dui').length || text.match('正确答案')) {
                text = text.match(/[A-G]/gi) || [];
                this.option = $.map(text, function (value) {
                    return filterImg($TiMu.find('.fl:contains(' + value + ') + a'));
                }).join('#') || '无';
                this.key = text.join('');
            } else if (this.code) {
                this.code = -1;
            } else {
                return false;
            }
        }
        return this;
    }), function (value) {
        return value && value.option != '无';
    });
    setting.curs = $('script:contains(courseName)', top.document).text().match(/courseName:\'(.+?)\'|$/)[1] || $('h1').text().trim() || '无';
    data.length && GM_xmlhttpRequest({
        method: 'POST',
        url: 'sea of flowers' + $('#workId').val(),
        headers: {
            'Content-type': 'application/x-www-form-urlencoded'
        },
        data: 'course=' + encodeURIComponent(setting.curs) + '&data=' + encodeURIComponent((Ext.encode || JSON.stringify)(data)) + '&id=' + $('#jobid').val().slice(5)
    });
    $job.addClass('ans-job-finished');
}

function filterImg(dom) {
    return $(dom).clone().find('img[src]').replaceWith(function () {
        return $('<p></p>').text('<img src="' + $(this).attr('src') + '">');
    }).end().find('iframe[src]').replaceWith(function () {
        return $('<p></p>').text('<iframe src="' + $(this).attr('src') + '"></irame>');
    }).end().text().trim();
}
