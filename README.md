NOTE: Following is the content of the livescript "CoronaSummarize.mlx"

pdf�t�@�C���̎擾�B���X�X�V����Ă����̂�ii�̒l�͑��₵�Ă����K�v����B���L��3/8���_�̂��́B�����ɂ�1���ԂقǗv���܂����B


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


COTOHA API���R�[�����鏀��


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


COTOHA API��p�����A�ŗL�\���ƃT�}���[�̎擾


```matlab
cTextMat = rmmissing(TextMat);

for ii = 1:length(cTextMat)
    try
        textNE{ii} = getNE(strtrim(cTextMat(ii)),tokens);
        textsum{ii} = getsum(cTextMat(ii),tokens,2);
    end
end
```


TAT��p�����ʒu���̎擾


```matlab
document = tokenDetails(tokenizedDocument(cTextMat));
locationdoc = [document.DocumentNumber(document.Entity == 'location'),document.Token(document.Entity == 'location')];
```


���t���̎擾


```matlab
for ii = 1:length(textNE)
    try
        dateinfo = struct2cell(textNE{ii});
        datedoc{ii} = dateinfo(3,strcmp(dateinfo(5,:),'DAT'));
    end
end
```


���ʂ̕\���Bdocnum�̐��l�X���C�_�[��ω������Ċm�F�B


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
Date = 1x4 �� cell �z��    
'2�N2�� 20��(��)'    '2��6��'       '2�� 17��'     '2�� 20��'     

```
```matlab
Location = locationdoc(strcmp(locationdoc(:,1),num2str(docnum)),2)
```
```
Location = 6x1 �� string �z��    
"����"         
"��"          
"����"         
"�L����"        
"�s"          
"��"          

```
```matlab
Summary = textsum{docnum}
```
```
Summary = '����ɂ�茧���Ŋm�F���ꂽ���Ґ��́A���v�łR���ƂȂ�܂����B�P�AႂȂ��ACT�ŗ����x����������A�ċz�s�S�Ȃ��A��b�\�B'
```
```matlab
Honbun = cTextMat(docnum)
```
```
Honbun = 
    "�ߘa�Q�N�Q�� 20���i�؁j
     ���ꌧ�ی���Õ�
     
     
     �i�S���j�n��ی��ی��j�����ǔ�
     
     
     �v���A���� �d�b 098-866-2215
     
     
     �V�^�R���i�E�C���X�����Ǌ��҂̔����ɂ��āi��R��j
     
     
     �ߘa�Q�N�Q�� 20 ���A���ꌧ���ɂ����āA�V���ɂP���̐V�^�R���i�E�C���X������
     ���҂��������܂����̂ł��m�点���܂��B
     
     
     ����ɂ�茧���Ŋm�F���ꂽ���Ґ��́A���v�łR���ƂȂ�܂����B
     
     
     �i�R��ځj
     
     
     �P ���ҏ��
     
     
     80��A�j���A�L����s�ݏZ
     
     
     �Q �E�ƁF�_��
     
     
     �R �o��
     
     
     �E�Q���U�� �F���M����A�P�Ȃ��A���׋C���B
     
     
     �E�Q�� 17�� �F���M 36.8���A���ӊ�����A��������A�H�~�ቺ�̂��ߓ��@�B�P�A
     ႂȂ��ACT�ŗ����x����������A�ċz�s�S�Ȃ��A��b�\�B
     
     
     �E�Q�� 20�� �F�m�F�����ɂėz���m�F
     
     
     �S ���݂̊��҂̏�
     
     
     �a��̑����Ȃ��A���ݓ��@��
     
     
     �T ���҂̍s��
     
     
     �ی����ɂĒ�����
     
     
     �U ���̑Ή�
     
     
     �ڐG�ғ��ɂ��ẮA�ی����ŏ����W��"

```


COTOHA API���R�[������֐�


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
