# Mobile Word Game Turn Server

<p>A simple HTTP TURN server used to demo my game's 2 player multiplayer. Can handle an undetermined number of clients, but the maximum number of pairing players, as well as the maximum number of games, is defaulted to 100.</p>

<p>Game clients connect to this server and the server handles matchmaking on a first-come-first-serve basis. Once clients are paired, this server relays data between the clients. This will not easily work with game engine net code since it's meant for simple text transfer. A game like mine which transfers very little data and doesn't require speedy, frequent communication will be a good fit.</p>
<p>How to use:</p>
<ul>
  <li><p>Clients send http POST requests to the server's public IP + whatever port is specified at startup (or default port is 80). The requests are simple text formatted like so:<blockquote>[incoming symbol],[formatted data],[game/pairing key],[player key],[game received switch]</blockquote></p>
    <ul>
      <li>[incoming symbol]: Tells the server what the client wants, and what kind of formatted data to expect. All of these symbols can be found under the comment reading "incoming symbols"</li>
       <li>[formatted data]: Game data related to the incoming symbol, or nothing. If the incoming symbol is NOTIFY_REGISTER, the data will be a user name. If the incoming symbol is NOTIFY_GAME_UPDATE, the data will be game data. Otherwise, this field is unused.</li>
      <li>[game/pairing key]: If the incoming symbol is NOTIFY_REGISTER(=0), the valid keys are '*' and 'bibbybabbis', which respectively give a normal and a long amount of time to wait between requests before being dropped. '*' is hard-coded, so the latter is for debug purposes. For any other incoming symbol, the key represents the unique pairing registry ID *or* game_registry ID supplied by the server.</li>
      <li>[player key]: Used only once the online match has started. Each player either sends a 0 or 1, uniquely. Used for list access in the Game object.</li>
      <li>[game recieved switch]: 0 or 1. Used to synchronize data between clients and the server in case of packet loss. The client keeps a boolean variable, initialized to 0. Every time a client recieves data from the other client (through this server), the boolean should flip.</li>
    </ul>
    <p>Example 1: <blockquote>"0,Davey Jones,*,*,*"</blockquote> being sent to the server says "Register me as Davey Jones. I need a key.". Example 2: <blockquote>"2,*,30,*,*"</blockquote> says "I want to be paired with another player. The registry key handed to me by the server is 30."
  </li>
  <li>Each request *should* get some kind of data in return. If no data is recieved, the server expects you to send the same data again. In the worst case, recieving most error codes indicates that the client must go back to step 1 and register.</li>
  <li>Data coming from the server starting with "0/" indicates success. The client is expected to change their incoming symbol depending on the data they get back from the server, so that the server knows the client agrees with it. For example, The client sends <blockquote>0,Martha Stewart,*,*,*</blockquote> - requesting to register. The server sends back "0/0/15", telling the client registration is successful and their key during the pairing process will be "15". So, the next request to the server should be <blockquote>2,*,15,*,*,*</blockquote>, requesting pairing for the client with key 15.</li>
  <li>The pairing process ends when the client gets a C_MSG_START_PLAY message. The message will come with a game key and the client's player key, which need to be supplied with all future requests.</li>
  <li>All formatted data going forward will be game data. The format is determined on the client end, because the server only passes the data along.</li>
  <li>command line arguments are detailed above __main__</p>
</ul>
