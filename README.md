# lark-hertz (This is a community driven project)

## Installation

```bash
go get github.com/hertz-contrib/lark-hertz
```

## Usage

```go
package main

import (
	"context"
	"fmt"

	"github.com/chyroc/lark"
	"github.com/cloudwego/hertz/pkg/app/server"
	"github.com/hertz-contrib/lark-hertz"
)

func main() {
	// 1: init lark client
	larkCli := lark.New(
		lark.WithAppCredential("", ""),
		lark.WithEventCallbackVerify("", ""),
	)

	// 2: register lark msg callback login
	// handle text, file, image and send response
	larkCli.EventCallback.HandlerEventV2IMMessageReceiveV1(func(ctx context.Context, cli *lark.Lark, schema string, header *lark.EventHeaderV2, event *lark.EventV2IMMessageReceiveV1) (string, error) {
		content, err := lark.UnwrapMessageContent(event.Message.MessageType, event.Message.Content)
		if err != nil {
			return "", err
		}
		switch event.Message.MessageType {
		case lark.MsgTypeText:
			_, _, err = cli.Message.Reply(event.Message.MessageID).SendText(ctx, fmt.Sprintf("got text: %s", content.Text.Text))
		case lark.MsgTypeFile:
			_, _, err = cli.Message.Reply(event.Message.MessageID).SendText(ctx, fmt.Sprintf("got file: %s, key: %s", content.File.FileName, content.File.FileKey))
		case lark.MsgTypeImage:
			_, _, err = cli.Message.Reply(event.Message.MessageID).SendText(ctx, fmt.Sprintf("got image: %s", content.Image.ImageKey))
		}
		return "", err
	})

	// 3: init hertz server
	h := server.Default()
	h.POST("/api/lark_callback", lark_hertz.ListenCallback(larkCli, lark_hertz.WithIgnoreCheckSignature(true)))

	h.Spin()

	// 4: deploy server to cloud, and set lark callback url to `<host>/api/lark_callback`
}
```

## License

This project is under Apache License. See the [LICENSE](./LICENSE-APACHE) file for the full license text.