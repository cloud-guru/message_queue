## Project layout

project nằm tại https://github.com/cloud-guru/message_queue

    mkdocs.yml                 # configuration file.
    docs/
        index.md               # Documentation homepage.
        issue/                 # Chứa các doc: 
          README.md            # Doc mô tả về vấn đề
          propose/             # Doc dự kiến thực hiện
          in_process/          # Doc đang thực hiện
          need_review/         # Doc đã hoàn thành, cần review
          sloved/              # Doc hoàn thành
        about/
          contributing.md      # Hướng dẫn cách đóng góp vào project
          license.md
          release-notes.md
          

## Commands 
Clone project: 

* `git clone https://github.com/cloud-guru/message_queue.git`

Install mkdocs:

* `pip install mkdocs`

Triển khai:

Clone và di chuyển vào project:

* `mkdocs gh-deploy` - deploy vào github . để có thể xem online (chỉ có user thuộc cloud-guru mới có thể làm)
* `mkdocs serve -a 0.0.0.0:8000 ` - Phục vụ xem local tại cổng 8000
* `mkdocs help` - Print  help.
xem thêm tại https://mkdocs.org