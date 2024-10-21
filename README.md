# golang-maroto-pdf-sample

This repository demo for implementing pdf generator with golang

## install package

```shell
go get github.com/johnfercher/maroto/v2@v2.2.1
```

## logo
```golang
func main() {
	// Initialize PDF with Portrait and A4 size
	cfg := config.NewBuilder().
		WithOrientation(orientation.Vertical).
		WithPageSize(pagesize.A4).
		WithLeftMargin(15).
		WithTopMargin(15).
		WithRightMargin(15).
		WithBottomMargin(15).
		Build()
	m := maroto.New(cfg)

	// Build sections of the PDF
	// 1. Header
	addHeader(m)
	// 2. Invoice Number
	addInvoiceDetails(m)
	// 3. Item List
	addItemList(m)
	// 4. Footer - Signature and QR code
	addFooter(m)
	// save to file
	document, err := m.Generate()
	if err != nil {
		log.Fatal(err.Error())
	}
	err = document.Save("output/invoice_sample.pdf")
	if err != nil {
		log.Fatal(err.Error())
	}
	log.Println("PDF save successfully")
}

func addHeader(m core.Maroto) {
	m.AddRow(50,
		image.NewFromFileCol(12, "assets/logo.png",
			props.Rect{
				Center:  true,
				Percent: 75,
			},
		),
	)
	m.AddRow(20,
		text.NewCol(12, "GSON",
			props.Text{
				Top:   5,
				Style: fontstyle.Bold,
				Align: align.Center,
				Size:  16,
			}),
	)
	m.AddRow(20,
		text.NewCol(12, "Invoice", props.Text{
			Top:   5,
			Style: fontstyle.Bold,
			Align: align.Center,
			Size:  12,
		}),
	)
}
func addInvoiceDetails(m core.Maroto) {
	m.AddRow(10,
		text.NewCol(6, "Date:"+time.Now().Format("20 Jan 2024"),
			props.Text{
				Align: align.Left,
				Size:  10,
			}),
		text.NewCol(6, "Invoice #1001",
			props.Text{
				Align: align.Right,
				Size:  10,
			}),
	)
	m.AddRow(10, line.NewCol(12))
}

type InvoiceItem struct {
	Item            string
	Description     string
	Quantity        string
	Price           string
	DiscountedPrice string
	Total           string
}

func (o InvoiceItem) GetHeader() core.Row {
	return row.New(10).Add(
		text.NewCol(2, "Item", props.Text{Style: fontstyle.Bold}),
		text.NewCol(3, "Description", props.Text{Style: fontstyle.Bold}),
		text.NewCol(1, "Quantity", props.Text{Style: fontstyle.Bold}),
		text.NewCol(2, "Price", props.Text{Style: fontstyle.Bold}),
		text.NewCol(2, "DiscountedPrice", props.Text{Style: fontstyle.Bold}),
		text.NewCol(2, "Total", props.Text{Style: fontstyle.Bold}),
	)
}
func (o InvoiceItem) GetContent(i int) core.Row {
	r := row.New(5).Add(
		text.NewCol(2, o.Item),
		text.NewCol(3, o.Description),
		text.NewCol(1, o.Quantity),
		text.NewCol(2, o.Price),
		text.NewCol(2, o.DiscountedPrice),
		text.NewCol(2, o.Total),
	)
	if i%2 == 0 {
		r.WithStyle(&props.Cell{
			BackgroundColor: &props.Color{Red: 240, Green: 240, Blue: 240},
		})
	}
	return r
}
func getObjects() []InvoiceItem {
	var items []InvoiceItem
	contents := [][]string{
		{"Laptop", "14-inch, 16GB RAM", "1", "$1200", "$1000", "$1000"},
		{"Mouse", "Wireless optical mouse", "2", "$25", "$20", "$40"},
		{"Keyboard", "Mechanical, RGB", "1", "$75", "$60", "$60"},
	}
	for i := 0; i < len(contents); i++ {
		items = append(items, InvoiceItem{
			Item:            contents[i][0],
			Description:     contents[i][1],
			Quantity:        contents[i][2],
			Price:           contents[i][3],
			DiscountedPrice: contents[i][4],
			Total:           contents[i][5],
		})
	}
	return items
}
func addItemList(m core.Maroto) {
	rows, err := list.Build[InvoiceItem](getObjects())
	if err != nil {
		log.Fatal(err)
	}
	m.AddRows(rows...)
}
func addFooter(m core.Maroto) {
	m.AddRow(15,
		text.NewCol(8, "Total Amount", props.Text{
			Top:   5,
			Style: fontstyle.Bold,
			Size:  10,
			Align: align.Right,
		}),
		text.NewCol(4, "$1100", props.Text{
			Top:   5,
			Style: fontstyle.Bold,
			Size:  10,
			Align: align.Center,
		}),
	)
	m.AddRow(40,
		signature.NewCol(6, "Authorized Signatory", props.Signature{
			FontFamily: fontfamily.Courier,
		}),
		code.NewQrCol(6, "https://github.com/leetcode-golang-classroom/golang-maroto-pdf-sample",
			props.Rect{
				Percent: 75,
				Center:  true,
			},
		),
	)
}
```

## TaskFile
```yaml
version: '3'

tasks:
  default:
    cmds:
      - echo "This is task cmd"
    silent: true
  
  build:
    cmds:
      - CGO_ENABLED=0 GOOS=linux go build -o bin/main cmd/main.go
    silent: true
  run:
    cmds:
      - ./bin/main
    deps:
      - build
    silent: true
```