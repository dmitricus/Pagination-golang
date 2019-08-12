# Pagination-golang
Пагинация на чистом golang

Реализация hendler: В модель передаем пагинатор 

paginationPages := util.Pagination(limit, all, linkLimit, start)  
limit     int //- Количество элементов на странице - 7  
all       int //- Общее количество элементов - 110  
linkLimit int //- Количество ссылок в состоянии - 5  
start     int //- Текущее смещение ( для первой страницы будет отсутствовать поэтому эту ситуацию нужно учесть)  
