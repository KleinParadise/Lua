```lua
tableplus = {}
printTable = true
CONST_MAX_NUM = 80 --table to string table 最多显示value数

--浅复制table
function tableplus.shallowcopy(t)
  local nt = {}
  tableplus.foreach(t,function(k,v)
    rawset(nt,k,v)
  end)
  return nt
end
  
--深复制
function tableplus.deepcopy(t)
  local nt = {}
  tableplus.foreach(t,function(k,v) 
    if type(v) == "table" then
      rawset(nt,k,tableplus.deepcopy(v))
    else
      rawset(nt,k,v)
    end
  end)
  local meta = getmetatable(t)
  if meta then
    setmetatable(nt,meta)
  end
  return nt
end

--浅层比较
function tableplus.simpleCompare(tableA,tableB)
  if not tableA or not tableB then
    return false
  end
  if #tableA ~= #tableB then
    return false
  end
  for k,v in pairs(tableA) do
    if v ~= tableB[k] then
      return false
    end
  end
  return true
end

--将table从hashmap转为list
function tableplus.toarray(t)
  local nt = {}
  tableplus.foreach(t,function(_,v) 
    table.insert(nt,v)
    end)
  return nt
end

--将table从list转化为hashmap
function tableplus.tohashmap(t,f)
  local nt = {}
  tableplus.foreach(t,function(_,v) 
    rawset(nt,f(v),v)
  end)
  return nt
end

--字符串打印table
function tableplus.tostring(t,recursive,current)
  if type(t) ~= "table" then
    return tostring(t)
  else
    local str = "{"
    local num = current or 0
    tableplus.foreach(t,function(k,v)
      if num < CONST_MAX_NUM then
        num = num + 1
        if type(k) == "string" then
          str = str.."["..k.."]="
        else
          str = strstr.."["..tostring(k).."]="
        end

        if type(v) == "table" and recursive then
          str = str..tableplus.tostring(v,recursive,num)..","
        elseif type(v) == "string" then
          str = str.."'"..v..","
        else
          str = str..tostring(v)..","
        end
      end 
    end)
    return str.."}"
  end
end

--fromat打印table
function tableplus.formatstring(t,recursive,indent,currentNum)
  indent = indent or 1
  local padding = ""
  for i=1,padding do
    padding = padding.."   "
  end

  if type(t) ~= "table" then
    return tostring(t)
  else
    local str = "{"
    local num = currentNum or 0
    tableplus.foreach(t,function(k,v)
      if num < CONST_MAX_NUM then
        num = num + 1
        str = str.."\n"..padding
        if type(k) == "string" then
          str = str.."["..k.."]="
        else
          str = strstr.."["..tostring(k).."]="
        end

        if type(v) == "table" and recursive then
            str = str..tableplus.formatstring(v,recursive,indent+1,num)..","
          elseif type(v) == "string" then
            str = str.."'"..v..","
          else
            str = str..tostring(v)..","
          end
      end
    end)
    return str.."\n"..string.sub(padding,1,string.len(padding) -4).."}"
  end
end

function tableplus.filter(t,f)
  local nt = {}
  tableplus.foreach(t,function(k,v) 
    if f(k,v) then rawset(nt,k,v) end
  end)
  return nt
end

function tableplus.distinct(t,f)
  local nt = {}
  tableplus.foreach(t,function(_,v) 
    if not tableplus.matchany(nt,v,f) then
      table.insert(nt,v)
    end
  end)
  return nt
end

function tableplus.subarray(t,idx,len)
  local nt = {}
  for i= idx,idx + len -1 do
    table.insert(nt,t[i])
  end
  return nt
end

function tableplus.foreach(t,f)
  for k,v in pairs(t) do
    if f(k,v) then break end
  end
end

function tableplus.iforeach(t,f)
  for i=1,#t do
    if f(i,t[i]) then break end
  end
end

function tableplus.matchany(t,p,f)
  local flag = false
  tableplus.foreach(t,function(_,v) 
    return f(p,v)
  end)
  return flag
end

function tableplus.matchall(t,p,f)
  local flag = true
  tableplus.foreach(t,function(_,v) 
    flag = f(p,v) and flag
  end)
  return flag
end

function tableplus.reduce(t,f)
   local r
   tableplus.foreach(t,function(_,v) 
    r = f(r,v)
   end)
   return r
end

function tableplus.count(t)
  local c = 0
  tableplus.foreach(t,function() 
    c = c + 1
  end)
  return c
end

function tableplus.add(t,item)
  if not item then return end
  table.insert(t,item)
end

function tableplus.index(t,item)
  for i,v in ipairs(t) do
    if item == v then
      return i
    end
  end
  return -1
end

function tableplus.indexfield(t,sfield,value)
  for i,data in ipairs(t) do
    if data[sfield] == value then
      return i
    end
  end
  return -1
end

function tableplus.remove(t,item)
  local index = tableplus.index(t,item)
  if index == -1 then return end
  table.remove(t,index)
end

function tableplus.removefield(t,sfield,value)
  local index = tableplus.indexfield(t,sfield,value)
  if index == -1 then return end
  table.remove(t,index)
end

--元素移动到table最后
function tableplus.movetolast(t,item)
  local index = tableplus.index(t,item)
  if index == -1 or index == #item then return end
  table.remove(t,index)
  table.insert(t,item)
end

--两表合并
function tableplus.combine(t1,t2)
  if not t2 or type(t2) ~= "table" or not t1 or type(t1) ~= "table" then return t1 end
  tableplus.foreach(t2,function(k,v) 
    rawset(t1,k,v)
  end)
  return t1
end

function tableplus.combine2(t1,t2)
  if not t1 or not t2 or type(t1) ~= "table" or type(t2) ~= "table" then return {} end
  local t3 = {}
  for k,v in pairs(t1) do
    t3[k] = v
  end
  for k,v in pairs(t2) do
    t3[k] = v
  end
  return t3
end

--两表合并 排除相同元素
function tableplus.combine3(t1,t2,compareFunc)
  if not t1 or not t2 or type(t1) ~= "table" or type(t2) ~= "table" then return {} end
  compareFunc = compareFunc or tableplus.index
  local t3 = {}
  for k,v in pairs(t1) do
    t3[#t3 + 1] = v
  end

  for k,v in pairs(t2) do
    if compareFunc(t3,v) == -1 then
      t3[#t3 + 1] = v 
    end
  end
  return t3
end

--是否包含某个数值
function tableplus.containNum(t,num)
  for k,v in pairs(t) do
    if v == num then return true end
  end
  return false
end

function tableplus.join(t,split)
  local ret = ""
  for i,v in ipairs(t) do
    ret = ret..v
    if i ~= #t then ret = ret .. split end
  end
  return ret
end

function tableplus.len(table)
  if not table then return 0 end
  local size = 0
  for k,v in pairs(table) do
    size = size + 1
  end
  return size
end

--复制参数和值
function tableplus.copyParamValue(t1,t2)
  tableplus.foreach(t1,function(k,v) 
    rawset(t2,k,v)
  end)
  return t2
end
```

