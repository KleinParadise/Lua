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
```

