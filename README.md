package main

import (
	"bytes"
	"encoding/json"
	"flag"
	"fmt"
	"io"
	"os"
)

func prettyPrint(r io.Reader, out io.Writer, indent string) error {
	var buf bytes.Buffer
	if _, err := buf.ReadFrom(r); err != nil {
		return err
	}
	raw := buf.Bytes()
	var obj interface{}
	if err := json.Unmarshal(raw, &obj); err != nil {
		return fmt.Errorf("invalid JSON: %w", err)
	}
	enc := json.NewEncoder(out)
	enc.SetIndent("", indent)
	enc.SetEscapeHTML(false)
	return enc.Encode(obj)
}

func main() {
	input := flag.String("in", "", "input file (omit for stdin)")
	outpath := flag.String("out", "", "output file (omit for stdout)")
	indent := flag.String("indent", "  ", "indent string (e.g. two spaces or \\t)")
	flag.Parse()

	var in io.Reader = os.Stdin
	if *input != "" {
		f, err := os.Open(*input)
		if err != nil {
			fmt.Fprintln(os.Stderr, "open error:", err); os.Exit(1)
		}
		defer f.Close()
		in = f
	}

	var out io.Writer = os.Stdout
	if *outpath != "" {
		f, err := os.Create(*outpath)
		if err != nil {
			fmt.Fprintln(os.Stderr, "create error:", err); os.Exit(1)
		}
		defer f.Close()
		out = f
	}

	if err := prettyPrint(in, out, *indent); err != nil {
		fmt.Fprintln(os.Stderr, err); os.Exit(2)
	}
}
