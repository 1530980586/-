# -
学习通自动签到

                audioPlayer = new Audio(audiofile),
                $ = $w.jQuery||$w.$,
                handleImgs = (s) => {
                    imgEs = s.match(/(<img([^>]*)>)/);
                    if (imgEs) {
                        for (let j = 0, k = imgEs.length; j < k; j++) {
                            let urls = imgEs[j].match(
                                    /http[s]?:\/\/(?:[a-zA-Z]|[0-9]|[$-_@.&+]|[!*\(\),]|(?:%[0-9a-fA-F][0-9a-fA-F]))+/),
                                url;
                            if (urls) {
                                url = urls[0].replace(/http[s]?:\/\//, '');
                                s = s.replaceAll(imgEs[j], url);
                            }
                        }
                    }
                    return s;
                },
                trim = (s) => {
                    return handleImgs(s).replaceAll('javascript:void(0);', '').replaceAll("&nbsp;", '').replaceAll("，", ',').replaceAll("。", '.').replaceAll("：", ':').replaceAll("；",';').replaceAll("？", '?').replaceAll("（", '(').replaceAll("）", ')').replaceAll("“", '"').replaceAll("”", '"').replaceAll("！", '!').replaceAll("-", ' ').replace(/(<([^>]+)>)/ig, '').replace(/^\s+/ig, '').replace(/\s+$/ig, '');
                };
            if(!classId||!courseId){
                alert('参数不全，请重新进入此页面');
                return;
            }
            audioPlayer.loop = true;
            $w.audioPlayer = audioPlayer;
            $d.addEventListener('visibilitychange', function() {
                var c = 0;
                if ($d.hidden) {
                    audioPlayer.play();
                    var timer = setInterval(function() {
                        if (c) {
                            $d.title = '🙈挂机中';
                            c = 0;
                        } else {
                            $d.title = '🙉挂机中';
                            c = 1;
                        }
                        if (!$d.hidden) {
                            clearInterval(timer);
                            $d.title = '超星学习通全能辅助';
                        }
                    }, 1300);
                } else {
                    audioPlayer.pause();
                }
            });
            $w.askedToken = false;
            $w.need = false;
            let pointList = [],page=1,pointNum=0;
            loopDetail:
            while(1){
                let hostUrl = $w.location.protocol+'//'+$w.location.host.replace(/mooc(.*?)\./ig,'stat2-ans.');
                if(!$w.location.host.includes('chaoxing.com')&&!$w.urlTold){
                    alert('接下来可能会弹出确认页面\n请在页面上选择“总是允许”或者“永久允许”\n否则脚本无法正常运行');
                    $w.urlTold = true;
                }
                let pointListResult = await request({
                    'headers':{
                        'Referer': hostUrl+'/stat2/task/s/index?courseid='+courseId+'&cpi='+cpi+'&clazzid='+classId+'&ut=s&'
                    },
                    'url':hostUrl+'/stat2/task/s/progress/detail?clazzid='+classId+'&courseid='+courseId+'&pageSize=16&page='+page+'&status=2'
                });
                if(pointListResult){
                    let pointListJson = JSON.parse(pointListResult.responseText);
                    if(!pointListJson.status){
                        break;
                    }
                    for(let i=0,l=pointListJson.data.results.length;i<l;i++){
                        pointNum+=pointListJson.data.results[i].list.length;
                        pointList.push(pointListJson.data.results[i]['id']);
                        if(pointList.length>=200){
                            $w.need = true;
                            logs.addLog('单次可循环200个任务点，任务点完成后将再次查询');
                            break loopDetail;
                        }
                    }
                    if(pointListJson.data.pageInfo.currentPageNo<pointListJson.data.pageInfo.totalPage){
                        page++;
                        await sleep(500);
                        continue;
                    }else{
                        break;
                    }
                }else if(pointList==[]){
                    alert('登录状态失效，请重新登陆超星');
                    $w.location.href = $w.location.protocol+'//'+$w.location.host.replace(/mooc(.*?)\./ig,'passport2.')+'/login?fid=&newversion=true&refer='+encodeURIComponent($w.location.href);
                    return;
                    
                }else{
                    break;
                }
            }
            $d.getElementById('totalNum').innerHTML = $d.getElementById('leftNum').innerHTML = pointNum;
            if(pointNum<1){
                logs.addLog('该课程无可用任务点，请检查，或尝试重新登录');
                return;
            }
            logs.addLog('共有'+pointNum+'个任务点');
            pointNum++;
            function *point(){
                for(let i=0,l=pointList.length;i<l;i++){
                    pointNum--;
                    $d.getElementById('leftNum').innerHTML = pointNum;
                    yield pointList[i];
                }
            }
            let getPoint = point();
            function getVideoEnc(item){
                let urlList = [],
                    playTime = 0,
                    isdrag='3',
                    continu = true; 
                while(continu){
                    if(playTime>item['duration']){
                        isdrag='4';
                        continu = false;
                        playTime=item['duration'];
                    }
                    let strEc = `[${classId}][${$uid}][${item['jobid']}][${item['objectId']}][${playTime * 1000}][d_yHJ!$pdA~5][${item['duration'] * 1000}][0_${item['duration']}]`,
                        enc = md5(strEc),
                        reportUrl = '/' + item['dtoken'] + '?clazzId=' + classId + '&playingTime=' + String(playTime) + '&duration=' + item['duration'] + '&clipTime=0_' + item['duration'] + '&objectId=' + item['objectId'] +'&otherInfo=' + item['otherInfo'] + '&jobid=' + item['jobid'] + '&userid=' + $uid + '&isdrag=' + isdrag + '&view=pc&enc=' + enc + '&rt=' + item['rt'] + '&dtype=' + item['dtype'] + '&_t=' + String(Math.round(new Date()));
                    playTime+=parseInt(60*倍速);
                    urlList.push([reportUrl,isdrag]);
                }
                return urlList;
            }
            loopPoint:
            while(1){
                let g = getPoint.next();
                if(g.done){
                    logs.addLog('所有任务已完成');
                    if($w.need){
                        logs.addLog('将进行第二次循环');
                        start();
                    }
                    break;
                }
                let chapterId = g.value,page=0;
                try{
                    let nowHour = new Date().getHours();
                    if((nowHour>=22||nowHour<7)&&!$w.told){
                        $w.told = true;
                        $w.layer.alert('夜间学习会导致学习进度被清空');
                    }
                }catch(e){console.log(e);}
                loopPage:
                while(1){
                    try{
                    while(1){
                        if(!$w.wait){
                            break;
                        }
                        await sleep(500);
                    }
                    await sleep(Math.random()*3000+1000);
                    let cardUrl = '/knowledge/cards?clazzid='+classId+'&courseid='+courseId+'&knowledgeid='+chapterId+'&num='+page+'&ut=s&cpi='+cpi+'&v=20160407-1',
                        Referer = $siteHost+'/mycourse/studentstudy?chapterId='+chapterId+'&courseId='+courseId+'&clazzid='+classId+'&cpi='+cpi+'&enc='+md5(chapterId)+'&mooc2=1&openc='+md5(chapterId+'p');
                        cardResult = await request({
                            'headers':{
                                'Referer': Referer 
                            },
                            'url':cardUrl
                        });
                    page++;
                    if(!cardResult){
                        continue loopPoint;
                    }
                    if(cardResult.responseText.includes('mArg = $mArg;')){
                        continue loopPoint;
                    }
                    let mArgs = cardResult.responseText.match(/mArg\s*=\s*(.+);\s*}\s*catch\(e\)/);
                    if(!mArgs){
                        logs.addLog('无法获取章节内容，章节可能未开放','red');
                        continue loopPoint;
                    }
                    let mArg = mArgs[0].replace(/mArg\s*=\s*/,'').replace(/;\s*}\s*catch\(e\)/,''),
                        mArgJson = JSON.parse(mArg),
                        reportUrl = mArgJson.defaults.reportUrl;
                    for(let i=0,l=mArgJson.attachments.length;i<l;i++){
                        try{
                        let jobData = mArgJson.attachments[i];
                        if(!jobData.job){
                            continue;
                        }
                        loopType:
                        switch(jobData.type){
                            case 'video':
                            if(!GM_getValue('doVideo',true)){
                                logs.addLog('跳过任务：'+jobData.property.name,'red');
                                break;
                            }
                            let statusUrl = '/ananas/status/' + jobData.property.objectid + '?k=' +
                    $fid + '&flag=normal&_dc=' + String(Math.round(new Date())),
                                videoInfoResult = await request({
                                    'headers':{
                                        'Referer': $w.location.protocol+'//'+$w.location.host+'/ananas/modules/pdf/index.html?v=2022-1202-1135&'
                                    },
                                    'url':statusUrl
                                });
                                if(!videoInfoResult){
                                    logs.addLog('获取视频信息失败，跳过此任务：'+jobData.property.name,'red');
                                    break loopType;
                                }
                                logs.addLog('开始刷视频：'+jobData.property.name);
                                let videojs_id = String(parseInt(Math.random() * 9999999));
                                $d.cookie = 'videojs_id=' + videojs_id + ';path=/'
                                let videoInfo = JSON.parse(videoInfoResult.responseText),
                                    videoDatas = getVideoEnc({
                                        'duration':videoInfo.duration,
                                        'jobid':jobData.jobid,
                                        'objectId':videoInfo.objectid,
                                        'dtoken':videoInfo.dtoken,
                                        'otherInfo':jobData.otherInfo,
                                        'rt':jobData.property.rt||'0.9',
                                        'dtype':jobData.property.module.includes('audio')?'Audio':'Video'
                                    }),
                                    errTimes = 0,
                                    min=0,
                                    totalMin = Math.round(videoInfo.duration/60);
                                for(let i=0,l=videoDatas.length;i<l;i++){
                                    let url = videoDatas[i][0],
                                        isdrag = videoDatas[i][1],
                                        sendUrl = reportUrl+url;
                                        watchResult = await request({
                                            headers: {
                                                'Referer': $w.location.protocol+'//'+$w.location.host+'/ananas/modules/video/index.html?v=2023-0303-1828',
                                                'Sec-Fetch-Site': 'same-origin',
                                                'Content-Type': 'application/json'
                                            },
                                            url:sendUrl
                                        });
                                    if(!watchResult){
                                        if(errTimes>2){
                                            logs.addLog('上传视频进度失败超过三次，跳过此任务：'+jobData.property.name,'red');
                                            break loopType;
                                        }
                                        errTimes++;
                                        logs.addLog('上传视频进度失败，将再次重试'+String(3-errTimes)+'次','red');
                                        continue;
                                    }else if(watchResult.status!='200'){
                                        if(errTimes>2){
                                            logs.addLog('上传视频进度失败超过三次，跳过此任务：'+jobData.property.name,'red');
                                            break loopType;
                                        }
                                        errTimes++;
                                        logs.addLog('超星返回错误信息，将再次重试'+String(3-errTimes)+'次','red');
                                        continue;
                                    }
                                    let watchResultJson = JSON.parse(watchResult.responseText);
                                    if(watchResultJson.isPassed){
                                        logs.addLog('视频任务完成：'+jobData.property.name,'green');
                                        break loopType;
                                    }else if(isdrag == '4'){
                                        logs.addLog('视频已观看完毕，但视频任务未完成：'+jobData.property.name,'red');
                                        break loopType;
                                    }else{
                                        倍速>1&&logs.addLog('开启倍速会被清空进度，当前倍速为'+倍速+'倍','red');
                                        logs.addLog('视频已观看'+min*倍速+'分钟，剩余大约'+String(totalMin-min*倍速)+'分钟');
                                    }
                                    min++;
                                    try{
                                        if(!$w.timeTold){
                                            let today = new Date(),
                                                todayStr = today.getFullYear() +'d' + today.getMonth() + 'd' + today.getDate(),
                                                timelong = GM_getValue('timelong', {});
                                            if (timelong.$uid == undefined ||timelong.$uid.today != todayStr){
                                                timelong.$uid = {
                                                    'time': 0,
                                                    'today': todayStr
                                                };
                                            } else {
                                                timelong.$uid.time++;
                                            }
                                            GM_setValue('timelong',timelong);
                                            if (timelong.$uid.time /60 >= 18 ) {
                                                $w.layer.alert('您今日学习时间过长，继续学习会清除进度');
                                                $w.timeTold = true;
                                            }
                                        }
                                    }catch(e){console.log(e);}
                                    await sleep(60000);
                                }
                            case 'document':
                                if(!GM_getValue('doDocument',true)){
                                    logs.addLog('跳过任务：'+jobData.property.name,'red');
                                    break;
                                }
                                logs.addLog('开始文档任务：'+jobData.property.name);
                                await sleep(5000);
                                let doDocumentResult = await request({'url':'/ananas/job/document?jobid=' + jobData.jobid +
                            '&knowledgeid=' + chapterId + '&courseid=' + courseId + '&clazzid=' + classId + '&jtoken=' + jobData.jtoken});
                                if(!doDocumentResult){
                                    logs.addLog('阅读文档失败：'+jobData.property.name,'red');
                                    break;
                                }
                                try{
                                    let doDocumentJson = JSON.parse(doDocumentResult.responseText);
                                    if(doDocumentJson.status){
                                        logs.addLog('文档任务完成：'+jobData.property.name,'green');
                                        break;
                                    }else{
                                        logs.addLog('文档任务失败：'+jobData.property.name,'red');
                                        break;
                                    }
                                }catch(e){
                                    logs.addLog('解析文档内容失败：'+jobData.property.name,'red');
                                    break;
                                }
                            case 'workid':
                                if(!GM_getValue('doWork',true)){
                                    logs.addLog('跳过任务：'+jobData.property.title,'red');
                                    break;
                                }
                                let workPageUrl = '/work/phone/work?workId=' + jobData.jobid.replace('work-', '') + '&courseId=' + courseId + '&clazzId=' + classId + '&knowledgeId=' + chapterId + '&jobId=' + jobData.jobid + '&enc=' + jobData.enc,
                                    workPageResult = await request({'url':workPageUrl});
                                if(!(workPageResult&&workPageResult.status==200&&workPageResult.responseText.includes('<title>章节</title>'))){
                                    logs.addLog('获取章节测试失败：'+jobData.property.title,'red');
                                    break;
                                }
                                let iframe = $d.getElementById('frame_content');
                                $('#workPanel').show();
                                iframe.setAttribute('srcdoc',workPageResult.responseText.replaceAll(/alert\(/ig,'console.log(').replaceAll(/confirm\((.*?)\)/ig,'true'));
                                await checkIframe(iframe);
                                let $p = $d.getElementById('frame_content').contentDocument,
                                    $pw = $d.getElementById('frame_content').contentWindow,
                                    wIdE = $p.getElementById('oldWorkId') || $p.getElementById('workLibraryId');
                                if(!wIdE){
                                    logs.addLog('获取章节测试错误：'+jobData.property.title,'red');
                                    $('#workPanel').hide();
                                    break;
                                }
                                let questionList = [],
                                    questionsElement = $p.getElementsByClassName('Py-mian1'),
                                    questionNum = questionsElement.length,//总题量
                                    abledQuestionNum = 0,//可作答的题量
                                    checkedQuestionNum = 0,//作答成功的题量
                                    wid = wIdE.value,
                                    optionElements = $p.getElementsByClassName('clearfix'),
                                    lis = $p.getElementsByTagName('li'),
                                    optionLis = [],
                                    answerInputs = $p.getElementsByClassName('answerInput');
                                if(!questionsElement.length){
                                    logs.addLog('无题：'+jobData.property.title,'red');
                                    $('#workPanel').hide();
                                    break;
                                }
                                for(let i=0,l=optionElements.length;i<l;i++){
                                    try{
                                    optionElements[i].setAttribute('class','clearfix');
                                    }catch(e){console.log(e);}
                                }
                                for(let i=0,l=lis.length;i<l;i++){
                                    try{
                                    if(lis[i].getAttribute('id-param')){
                                        optionLis.push(lis[i]);
                                    }
                                    }catch(e){console.log(e);}
                                }
                                for(let i=0,l=answerInputs.length;i<l;i++){
                                    try{
                                    answerInputs[i].value='';
                                    }catch(e){console.log(e);}
                                }
                                for (let i = 0; i < questionNum; i++) {
                                    try{
                                    let questionElement = questionsElement[i],
                                        idElements = questionElement.getElementsByTagName('input'),
                                        questionId = '0',
                                        question = questionElement.getElementsByClassName('Py-m1-title fs16')[0].innerHTML;
                                    question = handleImgs(question).replace(/(<([^>]+)>)/ig, '').replace(/[0-9]{1,3}.\[(.*?)\]/ig, '').replaceAll('\n','').replace(/^\s+/ig, '').replace(/\s+$/ig, '');
                                    for (let z = 0, k = idElements.length; z < k; z++) {
                                        try {
                                            if (idElements[z].getAttribute('name').indexOf('answer') >= 0) {
                                                questionId = idElements[z].getAttribute('name').replace('type', '');
                                                break;
                                            }
                                        } catch (e) {
                                            console.log(e);
                                            continue;
                                        }
                                    }
                                    if (questionId == '0' || question == '') {
                                        continue;
                                    }
                                    typeE = questionElement.getElementsByTagName('input');
                                    if (typeE == null || typeE == []) {
                                        continue;
                                    }
                                    let typeN = 'fuckme';
                                    for (let g = 0, h = typeE.length; g < h; g++) {
                                        if (typeE[g].id == 'answertype' + questionId.replace('answer', '').replace('check', '')) {
                                            typeN = typeE[g].value;
                                            break;
                                        }
                                    }
                                    if (['0', '1', '3'].indexOf(typeN) < 0) {
                                        continue;
                                    }
                                    type = {
                                        '0': '单选题',
                                        '1': '多选题',
                                        '3': '判断题'
                                    } [typeN];
                                    let optionList = {
                                        length: 0
                                    };
                                    if (['单选题', '多选题'].indexOf(type) >= 0) {
                                        let answersElements = questionElement.getElementsByClassName('answerList')[0].getElementsByTagName(
                                            'li');
                                        for (let x = 0, j = answersElements.length; x < j; x++) {
                                            let optionE = answersElements[x],
                                                optionTextE = trim(optionE.innerHTML.replace(/(^\s*)|(\s*$)/g, "")),
                                                optionText = optionTextE.slice(1).replace(/(^\s*)|(\s*$)/g, ""),
                                                optionValue = optionTextE.slice(0, 1);
                                            if (optionText == '') {
                                                break;
                                            }
                                            optionList[optionText]=optionValue
                                            optionList.length++;
                                        }
                                        if (answersElements.length != optionList.length) {
                                            continue;
                                        }
                                    }else{
                                        optionList = {'对':'A','错':'B'};
                                    }
                                    questionList.push({
                                        'question': question,
                                        'type': type,
                                        'questionid': questionId,
                                        'options': optionList
                                    });
                                    abledQuestionNum++;
                                    }catch(e){console.log(e);}
                                }
                                if(!abledQuestionNum){
                                    logs.addLog('该章节测试无可用题目：'+jobData.property.title,'red');
                                    $('#workPanel').hide();
                                    break;
                                }
                                logs.addLog('开始做章节测试：'+jobData.property.title);
                                if(!tkToken&&!$w.askedToken){
                                    await getToken('关注“一之哥哥”微信公众号，发送token获取您的题库token');
                                }
                                for(let i=0,l=questionList.length;i<l;i++){
                                    try{
                                    !i||await sleep(Math.random()*2000+2000);
                                    let questionArray = questionList[i],
                                        tm = questionArray.question,
                                        type = questionArray.type,
                                        options = questionArray.options,
                                        questionid = questionArray.questionid,
                                        wyzTkResult = false,
                                        wyzReady = false,
                                        trashTkResult = false,
                                        trashTkReady = false;
                                    if(tkToken){
                                        wyzTkResult = await request({
                                            headers: {
                                                'Content-type': 'application/x-www-form-urlencoded',
                                                'Authorization': tkToken,
                                            },
                                            url:'https://cx.icodef.com/wyn-nb?v=5',
                                            method: "POST",
                                            data: 'question=' + encodeURIComponent(tm)+'&type=' + {
                                                '单选题': '0',
                                                '多选题': '1',
                                                '判断题': '3'
                                            } [type] + '&id=' + wid,
                                        });
                                    }
                                    if(wyzTkResult){
                                        try{
                                            let wyzTkResultJson = JSON.parse(wyzTkResult.responseText);
                                            if(wyzTkResultJson.code!=1){
                                                if(wyzTkResultJson.msg){
                                                    logs.addLog('题库错误：'+wyzTkResultJson.msg);
                                                    if(wyzTkResultJson.msg.toLowerCase().includes('token')){
                                                        await getToken('题库错误：'+wyzTkResultJson.msg);
                                                    }
                                                }
                                            }else if(wyzTkResultJson.data){
                                                wyzReady = wyzTkResultJson.data;
                                            }
                                        }catch(e){
                                            console.log(e);
                                        }
                                    }
                                    if(!wyzReady){
                                        trashTkResult = await request({
                                            'url':host+'godofjs/tk/getAnswer.php?tm=' + encodeURIComponent(tm.replace(/(^\s*)|(\s*$)/g, '')) + '&type=' + {
                                                '单选题': '0',
                                                '多选题': '1',
                                                '判断题': '3'
                                            } [type] + '&wid=' + wid + '&courseid=' + courseId+'&version='+$version,});
                                        if(!trashTkResult){
                                            logs.addLog('未找到答案：'+tm);
                                            continue;
                                        }
                                        try{
                                            let trashTkResultJson = JSON.parse(trashTkResult.responseText);
                                            if(trashTkResultJson.code!=1){
                                                if(trashTkResultJson.msg){
                                                    logs.addLog('题库错误：'+trashTkResultJson.msg);
                                                    continue;
                                                }
                                            }else if(trashTkResultJson.data){
                                                trashTkReady = trashTkResultJson.data;
                                            }
                                        }catch(e){
                                            console.log(e);
                                        }
                                        if(!trashTkReady){
                                            logs.addLog('无答案：'+tm);
                                            continue;
                                        }
                                    }
                                    let tkRightAnswer = trim(wyzReady || trashTkReady);
                                    if(type=='判断题'){
                                        if('正确是对√Tri'.includes(tkRightAnswer)){
                                            tkRightAnswer = '对';
                                        }else if('错误否错×Fwr'.includes(tkRightAnswer)){
                                            tkRightAnswer = '错';
                                        }
                                    }
                                    logs.addLog(tm+'：'+tkRightAnswer);
                                    let hasAnswer = false;
                                    for(let o in options){
                                        if(o=='length'){
                                            continue;
                                        }
                                        o = trim(o);
                                        if(o.includes(tkRightAnswer)||tkRightAnswer.includes(o)){
                                            for(let j=0,a=optionLis.length;j<a;j++){
                                                let nowO = optionLis[j];
                                                if(nowO.getAttribute('id-param')==questionid.replace('answer','').replace('s','')){
                                                    if(nowO.getElementsByTagName('em')&&nowO.getElementsByTagName('em')[0].innerHTML==options[o]){
                                                        nowO.setAttribute('class','clearfix cur');
                                                        if(type=='判断题'){
                                                            $p.getElementById(questionid).value={'A':'true','B':'false'}[options[o]];
                                                        }else if(type=='单选题'){
                                                            $p.getElementById(questionid).value=options[o];
                                                        }else{
                                                            $p.getElementsByName(questionid)[0].value+=options[o];
                                                        }
                                                        hasAnswer = true;
                                                    }
                                                }
                                            }
                                        }
                                    }
                                    hasAnswer&&(checkedQuestionNum++);
                                    }catch(e){console.log(e);}
                                }
                                let score = checkedQuestionNum/questionNum*100;
                                if(score>=章节测试正确率&&!!GM_getValue('autoSubmit',1)){
                                    logs.addLog('正确率达标，自动提交');
                                    $pw.toadd();
                                    $pw.submitAction();
                                }else if(score>0){
                                    logs.addLog(['未设置自动提交','正确率不达标：'+String(Math.floor(score))+'分'][GM_getValue('autoSubmit',1)+0]+'，自动保存');
                                    $pw.noSubmit();
                                }else{
                                    logs.addLog('一道题都没查出来，跳过');
                                }
                                $('#workPanel').hide();
                                await sleep(Math.random()*2000+2000);
                                break;
                            default:
                                logs.addLog('暂不支持的任务类型：'+jobData.type);
                        }
                        }catch(e){
                            console.log(e);
                            try{logs.addLog('循环任务时出现无法预料的错误，请刷新页面重试，或联系作者反馈','red')}catch(e){}
                        }
                    }
                    }catch(e){
                        console.log(e);
                        try{logs.addLog('循环章节时出现无法预料的错误，请刷新页面重试，或联系作者反馈','red')}catch(e){}
                    }
                }
            }
        }
        main();
    }
})();

