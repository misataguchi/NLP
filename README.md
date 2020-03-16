NOTE: Following is the content of the livescript "CoronaSummarize.mlx"

pdfファイルの取得。日々更新されていくのでiiの値は増やしていく必要あり。下記は3/8時点のもの。処理には1時間ほど要しました。


```matlab
baseurl = 'https://www.mhlw.go.jp/content/10906000/000';

count = 1;
for ii = 596963:605192    
    basefname = 'Original';  
    url = [baseurl num2str(ii) '.pdf'];
    fname = [basefname num2str(ii) '.pdf'];
    try
        websave(fname,url);
    end
    if exist(fname)
        TextMat(count) = extractFileText(fname);
        delete(fname);
        count = count + 1;
    end
end

delete('*.html');
```


COTOHA APIをコールする準備


```matlab
clientid = '';
clientsecret = '';

url = 'https://api.ce-cotoha.com/v1/oauth/accesstokens';
options = weboptions('RequestMethod','post', 'MediaType','application/json');
Body = struct('grantType', 'client_credentials', ... 
          'clientId', clientid, ...
          'clientSecret', clientsecret);
tokens = webwrite(url, Body, options);
```


COTOHA APIを用いた、固有表現とサマリーの取得


```matlab
cTextMat = rmmissing(TextMat);

for ii = 1:length(cTextMat)
    try
        textNE{ii} = getNE(strtrim(cTextMat(ii)),tokens);
        textsum{ii} = getsum(cTextMat(ii),tokens,2);
    end
end
```


TATを用いた位置情報の取得


```matlab
document = tokenDetails(tokenizedDocument(cTextMat));
locationdoc = [document.DocumentNumber(document.Entity == 'location'),document.Token(document.Entity == 'location')];
```


日付情報の取得


```matlab
for ii = 1:length(textNE)
    try
        dateinfo = struct2cell(textNE{ii});
        datedoc{ii} = dateinfo(3,strcmp(dateinfo(5,:),'DAT'));
    end
end
```


結果の表示。docnumの数値スライダーを変化させて確認。


```matlab
docnum = 36
```
```
docnum = 36
```
```matlab
Date = datedoc{docnum}
```
```
Date = 1x4 の cell 配列    
'2年2月 20日(木)'    '2月6日'       '2月 17日'     '2月 20日'     

```
```matlab
Location = locationdoc(strcmp(locationdoc(:,1),num2str(docnum)),2)
```
```
Location = 6x1 の string 配列    
"沖縄"         
"県"          
"沖縄"         
"豊見城"        
"市"          
"県"          

```
```matlab
Summary = textsum{docnum}
```
```
Summary = 'これにより県内で確認された患者数は、合計で３名となりました。咳、痰なし、CTで両側肺炎所見あり、呼吸不全なし、会話可能。'
```
```matlab
Honbun = cTextMat(docnum)
```
```
Honbun = 
    "令和２年２月 20日（木）
     沖縄県保健医療部
     
     
     （担当）地域保健課結核感染症班
     
     
     久髙、岡野 電話 098-866-2215
     
     
     新型コロナウイルス感染症患者の発生について（第３報）
     
     
     令和２年２月 20 日、沖縄県内において、新たに１名の新型コロナウイルス感染症
     患者が発生しましたのでお知らせします。
     
     
     これにより県内で確認された患者数は、合計で３名となりました。
     
     
     （３例目）
     
     
     １ 患者情報
     
     
     80代、男性、豊見城市在住
     
     
     ２ 職業：農業
     
     
     ３ 経緯
     
     
     ・２月６日 ：微熱あり、咳なし、風邪気味。
     
     
     ・２月 17日 ：発熱 36.8℃、倦怠感あり、下痢あり、食欲低下のため入院。咳、
     痰なし、CTで両側肺炎所見あり、呼吸不全なし、会話可能。
     
     
     ・２月 20日 ：確認検査にて陽性確認
     
     
     ４ 現在の患者の状況
     
     
     病状の増悪なし、現在入院中
     
     
     ５ 患者の行動
     
     
     保健所にて調査中
     
     
     ６ 県の対応
     
     
     接触者等については、保健所で情報収集中"

```


COTOHA APIをコールする関数


```matlab
function textNE = getNE(sentence, tokens)

% Call COTOHA API
baseurl = 'https://api.ce-cotoha.com/api/dev/';
accesstoken = tokens.access_token;
Header = {'Content-Type', 'application/json;charset=UTF-8'; 'Authorization', ['Bearer ' accesstoken]};
% Body = struct('sentence', sentence);
Body = struct('sentence', sentence, ... 
          'type', 'kuzure',...
          'dic_type','medical');
options = weboptions('RequestMethod','post', 'MediaType','application/json','HeaderFields', Header,'Timeout',30);
response = webwrite([baseurl 'nlp/v1/ne'], Body, options);
textNE = response.result;

end

function textsum = getsum(sentence, tokens, sentlen)

% Call COTOHA API
baseurl = 'https://api.ce-cotoha.com/api/dev/';
accesstoken = tokens.access_token;
Header = {'Content-Type', 'application/json;charset=UTF-8'; 'Authorization', ['Bearer ' accesstoken]};
Body = struct('document', sentence, ... 
          'sent_len', sentlen);
options = weboptions('RequestMethod','post', 'MediaType','application/json','HeaderFields', Header,'Timeout',30);
response = webwrite([baseurl 'nlp/beta/summary'], Body, options);
textsum = response.result;

end
```
