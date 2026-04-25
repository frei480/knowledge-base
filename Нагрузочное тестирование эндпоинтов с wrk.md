---
tags:
  - python
---
# Случайный Поисковый Запрос В Wrk

## Пример Запуска Wrk Со Случайными (от 2 До 100)
Для запросов вида:
`http://localhost/api/videos/{id}/stream`
```bash
wrk -t12 -c100 -d30s -s <(cat <<'EOF'
ids={} for i=2,100 do table.insert(ids,i) end
setup=function() math.randomseed(os.time()) end
request=function()
  return wrk.format("GET", "/api/videos/"..ids[math.random(#ids)].."/stream")
end
EOF
) http://localhost
```

## Для Рандомизации Параметра `query` В `wrk`
Для запросов вида:
`http://localhost/api/search/?query={qs}`
```bash
wrk -t12 -c100 -d30s -s <(cat <<'EOF'
qs={"python","js","go","rust","lua"} math.randomseed(os.time())
request=function() 
return wrk.format("GET","/api/search/?query="..wrk.urlencode(qs[math.random(#qs)])) end
EOF
) http://localhost
```
