# Python LiveProject Summary
Python Live Project Summary

Over the past 2 weeks, I have been working on a team of other students to create a Hobby Tracker website application built with the Django framework. It was a great challenge and learning experience for me. I saw my strength and weaknesses as an aspiring software developer. This course taught me a lot about working on a live project utilizing DevOps, Agile and Scrum. This course also helped me practice and apply what I learned on the Python course. Since this is the first time I have ever worked on a project of this level I learned a lot about how developing teams work together and I became more familiar with Gitbash and Django framework. I think it is safe to say that I have a better understanding of how Django works but more importantly I learned what it is like to work with a team in a professional setting and I can take that knowledge and experience with me in all future endeavors as a software developer. This experience will help me further my knowledge and skills in the tech field.

# Lakers Scoreboard App
In this project, I built an app which retrieves an API response for the LA Lakers' final scoreboards. I also added an option in which the user may add, edit and delete a team's scoreboard and game dates.

## models.py

        class Scoreboard(models.Model):
            team1 = models.CharField(max_length=150)
            team2 = models.CharField(max_length=150)
            TeamScore1 = models.IntegerField()
            TeamScore2 = models.IntegerField()
            game_date = models.DateField(auto_now_add=False, auto_now=False, blank=True)

            Scoreboards = models.Manager()

            def __str__(self):
                return self.team1
                
## forms.py

        class DateInput(forms.DateInput):
            input_type = 'date'


        class ScoreboardForm(ModelForm):
            class Meta:
                model = Scoreboard
                fields = '__all__'
                widgets = {
                    'game_date': DateInput(),
                }

## views.py

        def home(request):
            return render(request, "LakersScoreboard/Scoreboard_home.html")


        def index(request):
            get_scores = Scoreboard.Scoreboards.all()
            context = {'scores': get_scores}
            return render(request, 'LakersScoreboard/Scoreboard_index.html', context)


        def add_scores(request):
            form = ScoreboardForm(request.POST or None)
            if form.is_valid():
                form.save()
                return redirect('listScores')
            else:
                print(form.errors)
                form = ScoreboardForm()
            return render(request, 'LakersScoreboard/Scoreboard_create.html', {'form': form})


        def details_score(request, pk):
            pk = int(pk)
            score = get_object_or_404(Scoreboard, pk=pk)
            context = {'score': score}
            return render(request, 'LakersScoreboard/Scoreboard_details.html', context)


        def edit_score(request, pk):
            pk = int(pk)
            score = get_object_or_404(Scoreboard, pk=pk)
            if request.method == "POST":
                form = ScoreboardForm(request.POST, instance=score)
                if form.is_valid():
                    score = form.save(commit=False)
                    score.save()
                    return redirect('detailsScoreboard', pk=score.pk)
            else:
                form = ScoreboardForm(instance=score)
            return render(request, 'LakersScoreboard/Scoreboard_edit.html', {'form': form})


        def delete_score(request, pk):
            pk = int(pk)
            score = get_object_or_404(Scoreboard, pk=pk)
            if request.method == "POST":
                score.delete()
                return redirect('listScores')
            context = {'score': score}
            return render(request, 'LakersScoreboard/Scoreboard_delete.html', context)


        def api_response(request):
            user_params = {'team_ids[]': 14, 'seasons[]': 2019, 'seasons[]': 2018, 'seasons[]': 2017, 'per_page': 100}
            response = requests.get('https://www.balldontlie.io/api/v1/games', params=user_params)
            json_response = json.loads(response.content)
            datalist = json_response['data']
            context = {"games": []}

            for item in datalist:
                unformatted_date = item.get('date')
                date = unformatted_date[0:10]
                home_team = item.get('home_team')
                home_team_name = home_team.get('full_name')
                home_team_score = item.get('home_team_score')
                visitor_team = item.get('visitor_team')
                visitor_team_name = visitor_team.get('full_name')
                visitor_team_score = item.get('visitor_team_score')
                scoreboard_dict = {
                    'date': date,
                    'home_team': home_team_name,
                    'home_team_score': home_team_score,
                    'away_name': visitor_team_name,
                    'away_team_score': visitor_team_score,
                }
                context["games"].append(scoreboard_dict)
            print(context)
            return render(request, 'LakersScoreboard/Scoreboard_api.html', context)

 
