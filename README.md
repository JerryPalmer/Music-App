# Music-App
from kivy.lang import Builder
from kivy.uix.screenmanager import ScreenManager, Screen
from kivy.uix.image import AsyncImage
from kivy.app import App
from googleapiclient.discovery import build

# Replace with your YouTube API key
YOUTUBE_API_KEY = "YOUR_YOUTUBE_API_KEY"

KV = '''
ScreenManager:
    HomeScreen:

<HomeScreen>:
    name: "home"
    BoxLayout:
        orientation: "vertical"

        # Background Image
        RelativeLayout:
            Image:
                source: "C:/Users/mew85240702/Desktop/Spotify.jpg"
                allow_stretch: True
                keep_ratio: False

        BoxLayout:
            orientation: "horizontal"
            size_hint_y: None
            height: "60dp"
            padding: [10, 10]
            spacing: 10

            TextInput:
                id: search_input
                hint_text: "Search for music..."
                size_hint_x: 0.8
                multiline: False
                padding: [10, 10]
                background_color: (0, 0, 0, 1)  # Black background
                foreground_color: (1, 1, 1, 1)  # White font

            Button:
                text: "Search"
                size_hint_x: 0.2
                background_color: (0, 0.5, 1, 1)
                on_release: root.search_music()

        AsyncImage:
            id: artist_image
            source: ""
            size_hint: (0.5, 0.5)
            pos_hint: {"center_x": 0.5, "center_y": 0.4}
            allow_stretch: True
            keep_ratio: True

        Label:
            id: now_playing_label
            text: "Now Playing: Nothing"
            size_hint_y: None
            height: "30dp"
            halign: "center"
            valign: "middle"
            color: (1, 1, 1, 1)  # White font for the label

        Label:
            id: artist_label
            text: "Artist: Unknown"
            size_hint_y: None
            height: "30dp"
            halign: "center"
            valign: "middle"
            color: (1, 1, 1, 1)  # White font for the label

        BoxLayout:
            orientation: "horizontal"
            size_hint_y: None
            height: "60dp"
            spacing: 20
            padding: [20, 10]

            Button:
                text: "Play"
                background_color: (0, 1, 0, 1)
                on_release: root.play_music()

            Button:
                text: "Stop"
                background_color: (1, 0, 0, 1)
                on_release: root.stop_music()

            Button:
                text: "Clear"
                background_color: (1, 1, 0, 1)
                on_release: root.clear_search()

        BoxLayout:
            orientation: "horizontal"
            size_hint_y: None
            height: "60dp"
            spacing: 20
            padding: [20, 10]

            Label:
                text: "Volume"
                size_hint_x: None
                width: "80dp"
                color: (1, 1, 1, 1)  # White font

            Slider:
                id: volume_slider
                min: 0
                max: 100
                value: 50
                size_hint_x: 0.8
                on_value: root.adjust_volume(self.value)
'''

class HomeScreen(Screen):
    def fetch_youtube_video(self, query):
        """Fetch video info from YouTube using the API."""
        youtube = build("youtube", "v3", developerKey=YOUTUBE_API_KEY)
        request = youtube.search().list(
            q=query,
            part="snippet",
            type="video",
            maxResults=1
        )
        response = request.execute()

        if response["items"]:
            video = response["items"][0]
            video_title = video["snippet"]["title"]
            channel_title = video["snippet"]["channelTitle"]
            thumbnail_url = video["snippet"]["thumbnails"]["high"]["url"]
            return video_title, channel_title, thumbnail_url
        return None

    def search_music(self):
        query = self.ids.search_input.text.strip()
        if query:
            self.ids.now_playing_label.text = f"Searching for: {query}"
            video_info = self.fetch_youtube_video(query)
            if video_info:
                video_title, channel_title, thumbnail_url = video_info
                self.ids.now_playing_label.text = f"Now Playing: {video_title}"
                self.ids.artist_label.text = f"Artist: {channel_title}"
                self.ids.artist_image.source = thumbnail_url
            else:
                self.ids.now_playing_label.text = "No results found."
                self.ids.artist_label.text = "Artist: Unknown"
                self.ids.artist_image.source = ""
        else:
            self.ids.now_playing_label.text = "Please enter a search term."
            self.ids.artist_label.text = "Artist: Unknown"
            self.ids.artist_image.source = ""

    def play_music(self):
        print("Playing music...")

    def stop_music(self):
        print("Stopping music...")

    def clear_search(self):
        self.ids.search_input.text = ""
        self.ids.now_playing_label.text = "Now Playing: Nothing"
        self.ids.artist_label.text = "Artist: Unknown"
        self.ids.artist_image.source = ""

    def adjust_volume(self, value):
        print(f"Volume adjusted to: {value}")


class MyMusicApp(App):
    def build(self):
        return Builder.load_string(KV)


if __name__ == "__main__":
    MyMusicApp().run()
