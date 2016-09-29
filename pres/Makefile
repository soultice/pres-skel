all: slides.html

clean:
	rm -f slides.html

slides.html: slides.md styles.css reveal.js
	pandoc -t revealjs -s slides.md -o slides.html --slide-level 2 \
	--css styles.css --template template.html --mathjax
