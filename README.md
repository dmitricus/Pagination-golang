# Pagination-golang
Пагинация на чистом golang

# Реализация Hendler передаем в шаблон модель и пагинатор
func ListOrdersHandler(config Config, m *model.Model) http.HandlerFunc {
	return func(w http.ResponseWriter, r *http.Request) {
		type Page struct {
			Orders                         []model.Order
			PaginationPages                []util.PaginationPage
			Next, Previous                 int
			NextIsActive, PreviousIsActive bool
		}
		// Получим первую и последнюю дату текущего года
		sm := util.DateYearGenerate()
		var (
			limit     = 7
			linkLimit = 5
			start     = 0
		)
		all, err := m.GetCountDateOrders(sm.StartDate, sm.EndDate)

		if err != nil {
			log.Printf("{\"error\":%q}", err.Error())
			return
		}
		if r.Method == "GET" {
			vars := mux.Vars(r)
			id := int(intVar(vars, "id"))
			if id != 0 {
				start = id
			}
		}
		paginationPages := util.Pagination(limit, all, linkLimit, start)
		log.Print("start: ", limit, all, linkLimit, start)

		orders, err := m.GetDateOrders(sm.StartDate, sm.EndDate, limit, start)
		if err != nil {
			log.Printf("{\"error\":%q}", err.Error())
			return
		}

		next := (start + limit)
		previous := (start - limit)
		previousIsActive := false
		nextIsActive := false
		if previous < 0 {
			previousIsActive = true
		}
		if next >= all {
			nextIsActive = true
		}

		page := Page{Orders: orders, PaginationPages: paginationPages, Next: next, Previous: previous, NextIsActive: nextIsActive, PreviousIsActive: previousIsActive}
		tmpl, err := template.ParseFiles(path.Join("assets/templates", "layout.html"), path.Join("assets/templates", "orders.html"))
		if err != nil {
			log.Printf("{\"error\":%q}", err.Error())
			return
		}

		if err := tmpl.ExecuteTemplate(w, "layout", page); err != nil {
			log.Println(err.Error())
			http.Error(w, http.StatusText(500), 500)
		}
	}
}
