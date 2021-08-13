- 👋 Hi, I’m @marinegreb
- 👀 I’m interested in ...
- 🌱 I’m currently learning ...
- 💞️ I’m looking to collaborate on ...
- 📫 How to reach me ...

<!---
marinegreb/marinegreb is a ✨ special ✨ repository because its `README.md` (this file) appears on your GitHub profile.
You can click the Preview link to take a look at your changes.
--->

package main

import (
	"bytes"
	"fmt"
	_ "image/jpeg"
	_ "image/png"
	"io"
	"log"
	"os"
	"text/template"
	"time"

	"github.com/skip2/go-qrcode"
)

func main() {
	a := &Authorization{
		CreateDate:        myTime{time.Now()},
		Name:              "Doe",
		FirstName:         "John",
		BirthDate:         myTime{time.Now()},
		BirthCity:         "Paris",
		Address:           "1 rue de Paris 75000 Paris",
		AuthorizationDate: myTime{time.Now()},
		Purpose:           "demarche",
	}
	err := a.Png(os.Stdout)
	if err != nil {
		log.Fatal(err)
	}

}

type Authorization struct {
	CreateDate        myTime
	Name              string
	FirstName         string
	BirthDate         myTime
	BirthCity         string
	Address           string
	AuthorizationDate myTime
	/* Purpose is one of
	"travail"
	"sante"
	"famille"
	"handicap"
	"judiciaire"
	"missions"
	"transit"
	"animaux"
	"courses"
	"sport"
	"rassemblement"
	"demarche"
	// source https://gitlab.inria.fr/stopcovid19/stopcovid-android/-/blob/master/stopcovid/src/main/assets/Attestations/form.json
	*/
	Purpose string
}

const tmpl = `Cree le: {{ .CreateDate.Date }} a {{ .CreateDate.HourMinute }};
Nom: {{ .Name }};
Prenom: {{ .FirstName }};
Naissance: {{ .BirthDate.Date }} a {{ .BirthCity }};
Adresse: {{ .Address }};
Sortie: {{ .AuthorizationDate.Date }} a {{ .AuthorizationDate.HourMinute }};
Motifs: {{ .Purpose }}`

func (a *Authorization) String() string {
	var b bytes.Buffer
	t := template.Must(template.New("attestation").Parse(tmpl))
	err := t.Execute(&b, a)
	if err != nil {
		panic(err)
	}
	return b.String()
}

func (a *Authorization) Png(w io.Writer) error {
	qrCode, err := qrcode.New(a.String(), qrcode.Medium)
	if err != nil {
		return err
	}
	return qrCode.Write(256, w)
}

type myTime struct {
	time.Time
}

var months = [...]string{
	"janvier",
	"fevrier",
	"mars",
	"avri",
	"mai",
	"juin",
	"juillet",
	"aout",
	"septembre",
	"octobre",
	"novembre",
	"decembre",
}

func (m myTime) Date() string {
	return fmt.Sprintf("%v %v %v", m.Format("02"), months[m.Month()-1], m.Format("2006"))
}

func (m myTime) HourMinute() string {
	return m.Format("15:04")
}
