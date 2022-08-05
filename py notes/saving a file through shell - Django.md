# saving a file through shell - Django
```python
def class Paper(models.Model):
    paper_id = models.CharField(primary_key=True, max_length=20)
    user_id = models.ForeignKey(UserData, on_delete=models.CASCADE)
    paper_title = models.TextField(max_length=200, validators=[validate_paper_title])
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True, null=True)
    paper_keywords = models.TextField(max_length=400)
    docfile = models.FileField(
        upload_to=get_doc_path, validators=[FileExtensionValidator(["pdf", "docx"])]
    )
    final_paper = models.FileField(
        upload_to=get_final_doc_path,
        validators=[FileExtensionValidator(["pdf"])],
        null=True,
    )
    pptfile = models.FileField(
        upload_to=get_ppt_path,
        null=True,
        blank=True,
        validators=[FileExtensionValidator(["pdf", "ppt", "pptx"])],
    )
    paper_status = models.CharField(
        max_length=50,
        choices=PaperStatusChoices.choices,
        default=PaperStatusChoices.UPLOADED,
    )
    comments = models.TextField(max_length=1000, null=True, blank=True)
    speaker = models.ForeignKey(
        UserData, on_delete=models.CASCADE, related_name="speaker", null=True
    )
    
    def __str__(self):
        return self.paper_title
```

and in the shell

```python
from django.core.files import File
paper = Paper()
paper.paper_id = '0001'
paper.user_id = user_inst
f = File(open('<path to the file>', 'rb')) # open the file you want to save
paper.docfile.save('<give a name for the file'>, f) # pass the file here and save it
paper.save() # save the instance
```
