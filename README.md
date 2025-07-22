# Simcenter-Simulation-Driver
MATLAB code for secondary development of Simcenter. Users can run simulations by modifying parameters in .txt files. The script loads models, sets simulation options, runs the solver, and outputs results automatically.

clc;clear;

addpath(fullfile(getenv('AME'),'scripting','matlab','amesim'));%将AMESim专用函数添加到运行路径

try

% 读取仿真文件的文件名

name = read_data('20250120_CXY_To_Xie输入参数/input_amefile.txt','ameFileName');

% 检查模型（文件名与模型名称必须一致）

eval(['! AMECirChecker -g -q --nobackup --nologfile ',name])

% 载入模型（文件名与模型名称必须一致）

eval([' ! AMELoad ', name])

sname1 = split(name,'.');

sname= char(sname1(1));

% 设置仿真参数

%读取仿真参数

f=read_data('20250120_CXY_To_Xie输入参数/算法参数.txt','finalTime');

s=read_data('20250120_CXY_To_Xie输入参数/算法参数.txt','step');

q=cell2mat(f);

% 设置仿真参数

sim_opt = amegetsimopt(sname);

sim_opt.finalTime = str2double(cell2mat(f));

sim_opt.printInterval = str2double(cell2mat(s));

%处理输出参数

[r,s]=ameloadt(name);

output=strings(size(s,1),1);

s6 = strings(size(s,1),1);

for i=1:size(s,1)
    
    a=strsplit(s(i,:),']');
    
    a=[cell2mat(a(1)),']'];
    
    s6(i)=string(a);

end

for i=2:size(s6,1)

inputStr = char(s6(i));

formattedStr = formatString1(inputStr);

s6(i) = string(formattedStr);

end

for i=1:size(s6,1)
    
    s6(i)=erase(s6(i),'[');
    
    s6(i)=erase(s6(i),']');

end

for i=1:size(s6,1)
    
    s6(i)=strrep(s6(i),' ','#');

end

s6(1)=[];

%判断是否有上一次仿真

if exist('20250120_CXY_To_Xie输入参数/nextinput_Param.txt', 'file')

%读取上一次仿真的输出以及用户需要修改的量

canshu_ming=read_data('nextinput_Param.txt','nextinput_Param_ming');

canshu_zhi_str=strings(size(canshu_ming));

canshu_zhi_str=read_data('nextinput_Param.txt','nextinput_Param_zhi');

for i=1:size(canshu_ming,1)
    
    canshu_zhi_double(i,:)=str2double(canshu_zhi_str(i));

end

canshu_danwei = read_data('nextinput_Param.txt','nextinput_Param_danwei');

gain_parname = read_data('20250120_CXY_To_Xie输入参数/参数化变量.txt','paramName');

for i=1:size(gain_parname,1)
    
    gain_parname(i)= strrep(gain_parname(i),'.',';');

end

%找出用户指定输入与上一次仿真重合的量

gain_parname = setdiff(gain_parname,canshu_ming);

for i=1:size(gain_parname,1)
    
    gain_parname(i)= strrep(gain_parname(i),';','.');

end

%修改上一次仿真输出的格式

a=' [';

b=']';

for i=1:size(canshu_ming,1)
    canshu_ming(i)= strrep(canshu_ming(i),'#',' ');
    canshu_ming(i)=strrep(canshu_ming(i),'_',' instance ');
    canshu_ming(i)=strrep(canshu_ming(i),',',' ');
    canshu_ming(i)=strrep(canshu_ming(i),';',' ');
    canshu_danwei(i)=strcat(a,canshu_danwei(i));
    canshu_danwei(i)=strcat(canshu_danwei(i),b);
    canshu(i) = strcat(canshu_ming(i),' ',canshu_danwei(i));
end
canshu = canshu';
canshu1 = strings(size(canshu,1),1);
for i=1:size(canshu,1)
    canshu1(i,:)=cell2mat(canshu(i));
end
end
if ~exist('nextinput_Param.txt', 'file')    
    gain_parname = read_data('20250120_CXY_To_Xie输入参数/参数化变量.txt','paramName');
end
%读取需要修改的值
n=strings(size(gain_parname));
for i=1:size(gain_parname,1)
    n(i)=read_data('20250120_CXY_To_Xie输入参数/参数化变量.txt',gain_parname(i));
    m(i,:)=str2double(n(i));
end
 for i=1:size(gain_parname,1)
    gain_parname(i)= strrep(gain_parname(i),'.',';');
    end
    
%读取input文件
filename = '20250120_CXY_To_Xie输入参数/ame_inputParam.txt';
fileID = fopen(filename, 'r');
if fileID == -1
    error('Cannot open file: %s', filename);
end
textData = textscan(fileID, '%s', 'Delimiter', '\n');
fclose(fileID);
%寻找参数对应的单位
tokens=cell(size(gain_parname));
a=' [';
b=']';
for i=1:size(gain_parname,1)
    keywords = gain_parname(i);
lineIndex = find(contains(textData{1}, keywords));
if ~isempty(lineIndex)
    lineContent = textData{1}{lineIndex};
    lineContent = lineContent(1:end-1);
    % 提取最后一个单位（假设单位是字母字符并且没有数字）
    tokens(i) = regexp(lineContent, '[^;]+$', 'match');
    if ~isempty(tokens(i))
        tokens(i)=strcat(a,tokens(i));
        tokens(i)=strcat(tokens(i),b);
    else
        tokens(i)='0';
    end
 end
end
%修改txt中读取的参数名称
for i=1:size(gain_parname,1)
    gain_parname(i)= strrep(gain_parname(i),'#',' ');
end
%将单位与参数组合起来
for i=1:size(gain_parname,1)
    gain_parname(i)=strrep(gain_parname(i),'_',' instance ');
    gain_parname(i)=strrep(gain_parname(i),',',' ');
    gain_parname(i)=strrep(gain_parname(i),';',' ');
   gain_parname(i)=strcat(gain_parname(i),tokens(i));
end
gain_parname1 = strings(size(gain_parname,1),1);
for i=1:size(gain_parname,1)
    gain_parname1(i,:)=cell2mat(gain_parname(i));
end
%修改参数
for i=1:size(gain_parname,1)
    ameputp(sname,char(gain_parname1(i)),m(i));
end
if exist('canshu', 'var')
    for i=1:size(canshu,1)
    ameputp(sname,char(canshu1(i)),canshu_zhi_double(i));
    end
end
%读取用户想要输出的参数
displacement_varname=read_data('20250120_CXY_To_Xie输入参数/output_yonghu.txt','output_yonghu');
displacement_varname=transpose(displacement_varname);

%修改参数格式
for i=1:size(displacement_varname,1)
    displacement_varname(i)= strrep(displacement_varname(i),'.',';');
end
%读取output_param,txt文件
filename = '20250120_CXY_To_Xie输入参数/ame_outputParam.txt';
fileID = fopen(filename, 'r');
if fileID == -1
    error('Cannot open file: %s', filename);
end
textData = textscan(fileID, '%s', 'Delimiter', '\n');
fclose(fileID);
%寻找参数对应的单位
tokens=cell(size(displacement_varname));
a=' [';
b=']';
for i=1:size(displacement_varname,1)
    keywords = displacement_varname(i);
lineIndex = find(contains(textData{1}, keywords));
if ~isempty(lineIndex)
    lineContent = textData{1}{lineIndex};
    lineContent = lineContent(1:end-1);
    % 提取最后一个单位（假设单位是字母字符并且没有数字）
    tokens(i) = regexp(lineContent, '[^;]+$', 'match');
    if ~isempty(tokens(i))
        tokens(i)=strcat(a,tokens(i));
        tokens(i)=strcat(tokens(i),b);
    else
        tokens(i)='0';
    end
 end
end
%修改参数格式
for i=1:size(displacement_varname,1)
    displacement_varname(i)= strrep(displacement_varname(i),';',',');
    displacement_varname(i)= strrep(displacement_varname(i),'#',' ');
end
for i=1:size(displacement_varname,1)
displacement_varname0(i)=displacement_varname(i);  
displacement_varname0(i)= strrep(displacement_varname0(i),' ','#');
displacement_varname0(i)= strrep(displacement_varname0(i),',','.');
end
%将单位与参数组合
for i=1:size(displacement_varname,1)
    displacement_varname(i)=strrep(displacement_varname(i),',',' ');
   displacement_varname(i)=strcat(displacement_varname(i),tokens(i));
   displacement_varname0(i)=strrep(displacement_varname0(i),',',' ');
   displacement_varname0(i)=strcat(displacement_varname0(i),tokens(i));
   displacement_varname0(i)=strrep(displacement_varname0(i),' ','');
end
displacement_varname1 = strings(size(displacement_varname,1),1);
for i=1:size(displacement_varname,1)
    displacement_varname1(i,:)=cell2mat(displacement_varname(i));
end
for i=1:size(displacement_varname,1)
displacement_varname0(i)= strrep(displacement_varname0(i),'[','{');
displacement_varname0(i)= strrep(displacement_varname0(i),']','}');
end
% 仿真运行（模型名称，模型仿真条件）
amerunsingle(sname, sim_opt);
amegetp(sname)
% 仿真结果获取
[results,var_names] = ameloadt(sname); % 获取所有参数名称+结果
for i=1:size(results,1)
next_input(i,:) = results(i,end);
end
for i=1:size(displacement_varname,1)
displacement_data(i,:) = amegetvar(results,var_names,char(displacement_varname(i)));% 查找自己需要的结果 （results,var_names，输出变量名称）
end
%输出用户所需的仿真结果
fid_out=fopen('20250120_CXY_To_Xie输入参数/finaloutput.txt','w');

% 手动写时间的表头
fprintf(fid_out, '%%ComArray 时间  STR %d %d %%\n', size(results(1,:)',1), size(results(1,:)',2));
fprintf(fid_out, 'time\n');  % 关键：表头下方写time

% 写时间数据（逐行）
for j=1:size(results(1,:)',1)
    fprintf(fid_out, '%.6e\n', results(1,j));   % 注意格式
end
fprintf(fid_out, '%%end%%\n');

% 其他变量继续按write_data写
for i=1:size(displacement_varname,1)
    output_tag = sprintf('Output%d', i);   % 构造Output1、Output2、...
    var_real_name = cell2mat(displacement_varname0(i));
    write_data(fid_out, output_tag, '%ComArray', 'STR', string(displacement_data(i,:)'), var_real_name);
end
fclose(fid_out);
%组合输出与参数
for i=1:size(s6,1)
    str = num2str(results(i+1,end));
    next_canshu=strcat(s6(i),';',str);
    s7(i,:) = next_canshu;
end
%输出带值的输出参数
fid_out=fopen('20250120_CXY_To_Xie输入参数/ame_outputParam1.txt','w');
write_data(fid_out,'20250120_CXY_To_Xie输入参数/outPutParamName', '%array2','STR', string(s7));
fclose(fid_out);
fclose('all');

catch ME
   fid_out=fopen('baocuo.txt','w');
   str = sprintf('仿真失败：%s', ME.message);
   arr = [str];
   write_data(fid_out,'error', '%array2','STR',arr);
end
function formattedStr = formatString(inputStr, value)
instancePos = strfind(inputStr, 'instance');
if isempty(instancePos)
error('输入字符串的格式不正确');
end
numStart = instancePos + length('instance');
instancePart = inputStr(1:numStart + 1);
unitStart = strfind(inputStr, '[');
if isempty(unitStart)
namePart = strtrim(inputStr(numStart + 3:end));
instancePart = strrep(instancePart, ' instance ', '_');
formattedStr = sprintf('%s;%s;%d', instancePart, namePart, value);
else
namePart = strtrim(inputStr(numStart + 3:unitStart-1));
unitPart = inputStr(unitStart:end);
instancePart = strrep(instancePart, ' instance ', '_');
formattedStr = sprintf('%s;%s;%d;%s', instancePart, namePart, value, unitPart);
end
end
function formattedStr = formatString1(inputStr)
instancePos = strfind(inputStr, '_');
if isempty(instancePos)
error('输入字符串的格式不正确');
end
numStart = instancePos + length('_');
instancePart = inputStr(1:numStart);
unitStart = strfind(inputStr, '[');
if isempty(unitStart)
namePart = strtrim(inputStr(numStart + 2:end));
formattedStr = sprintf('%s;%s;%d', instancePart, namePart, value);
else
namePart = strtrim(inputStr(numStart + 2:unitStart-1));
unitPart = inputStr(unitStart:end);
formattedStr = sprintf('%s;%s;%s', instancePart, namePart, unitPart);
end
end
