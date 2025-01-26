
First of all you need to install plugin     
npm install react-native-audio-recorder-player

Add this permission in AndroidManifest.xml file
<uses-permission android:name="android.permission.RECORD_AUDIO"/>
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"/>
<uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE"/>


Then add this functionality
import React, {useState, useRef} from 'react';
import {
  View,
  Text,
  Alert,
  TouchableOpacity,
  StyleSheet,
  PermissionsAndroid,
  Platform,
  FlatList
} from 'react-native';
import AudioRecorderPlayer from 'react-native-audio-recorder-player';
import RNFS from 'react-native-fs';

const App = () => {
  const [recordSecs, setRecordSecs] = useState(0);
  const [recordTime, setRecordTime] = useState('00:00:00');
  const [currentPositionSec, setCurrentPositionSec] = useState(0);
  const [playTime, setPlayTime] = useState('00:00:00');
  const [duration, setDuration] = useState('00:00:00');
  const [audioPath, setAudioPath] = useState('');
  const [fileList, setFileList] = useState([]);

  const audioRecorderPlayer = useRef(new AudioRecorderPlayer()).current;

  const requestPermissions = async () => {
    if (Platform.OS === 'android') {
      try {
        const granted = await PermissionsAndroid.requestMultiple([
          PermissionsAndroid.PERMISSIONS.RECORD_AUDIO,
          PermissionsAndroid.PERMISSIONS.WRITE_EXTERNAL_STORAGE,
          PermissionsAndroid.PERMISSIONS.READ_EXTERNAL_STORAGE,
        ]);

        if (
          granted['android.permission.RECORD_AUDIO'] !==
            PermissionsAndroid.RESULTS.GRANTED ||
          granted['android.permission.WRITE_EXTERNAL_STORAGE'] !==
            PermissionsAndroid.RESULTS.GRANTED ||
          granted['android.permission.READ_EXTERNAL_STORAGE'] !==
            PermissionsAndroid.RESULTS.GRANTED
        ) {
          Alert.alert(
            'Permission Denied',
            'Please grant all permissions to use this feature.',
          );
          return false;
        }
        return true;
      } catch (err) {
        console.warn(err);
        return false;
      }
    }
    return true; // iOS permissions are handled via Info.plist
  };

  const onStartRecord = async () => {
    const hasPermission = await requestPermissions();
    if (!hasPermission) return;

    try {
      const path = `${RNFS.DocumentDirectoryPath}/${Math.random()
        .toString(36)
        .substring(7)}.mp3`;
      setAudioPath(path);
      await audioRecorderPlayer.startRecorder(path);

      audioRecorderPlayer.addRecordBackListener(e => {
        setRecordSecs(e.current_position);
        setRecordTime(
          audioRecorderPlayer.mmssss(Math.floor(e.current_position)),
        );
      });
    } catch (error) {
      console.error('Start Record Error:', error);
    }
  };

  const onStopRecord = async () => {
    try {
      await audioRecorderPlayer.stopRecorder();
      audioRecorderPlayer.removeRecordBackListener();
      setRecordSecs(0);
    } catch (error) {
      console.error('Stop Record Error:', error);
    }
  };

  const onStartPlay = async () => {
    if (!audioPath) {
      Alert.alert('No Recording', 'Please record audio first.');
      return;
    }

    try {
      await audioRecorderPlayer.startPlayer(audioPath);

      audioRecorderPlayer.addPlayBackListener(e => {
        setCurrentPositionSec(e.current_position);
        setPlayTime(audioRecorderPlayer.mmssss(Math.floor(e.current_position)));
        setDuration(audioRecorderPlayer.mmssss(Math.floor(e.duration)));
      });
    } catch (error) {
      console.error('Play Error:', error);
    }
  };

  const onStopPlay = async () => {
    try {
      await audioRecorderPlayer.stopPlayer();
      audioRecorderPlayer.removePlayBackListener();
    } catch (error) {
      console.error('Stop Play Error:', error);
    }
  };

  const fetchAudioFiles = async () => {
    try {
      const files = await RNFS.readDir(RNFS.DocumentDirectoryPath);
      // const audioFiles = files.filter(file => file.name === 'cnusb.mp3');
      const audioFiles = files.filter(file => file.name.endsWith('.mp3'));
      setFileList(audioFiles);
    } catch (error) {
      console.error('Fetch Audio Files Error:', error);
    }
  };

  const renderItem = ({item}) => (
    <TouchableOpacity
      style={styles.audioItem}
      onPress={() => onStartPlay(item.path)}>
      <Text style={styles.audioText}>{item.name}</Text>
    </TouchableOpacity>
  );

  return (
    <View style={styles.container}>
      <Text style={styles.title}>Audio Recorder Player</Text>
      <Text style={styles.timer}>{recordTime}</Text>
      <View style={styles.buttonGroup}>
        <TouchableOpacity style={styles.button} onPress={onStartRecord}>
          <Text style={styles.buttonText}>Record</Text>
        </TouchableOpacity>
        <TouchableOpacity style={styles.button} onPress={onStopRecord}>
          <Text style={styles.buttonText}>Stop</Text>
        </TouchableOpacity>
      </View>

      <View style={styles.divider} />

      <Text style={styles.timer}>
        {playTime} / {duration}
      </Text>
      <View style={styles.buttonGroup}>
        <TouchableOpacity style={styles.button} onPress={onStartPlay}>
          <Text style={styles.buttonText}>Play</Text>
        </TouchableOpacity>
        <TouchableOpacity style={styles.button} onPress={onStopPlay}>
          <Text style={styles.buttonText}>Stop</Text>
        </TouchableOpacity>
      </View>

      <TouchableOpacity style={styles.button} onPress={fetchAudioFiles}>
        <Text style={styles.buttonText}>My List</Text>
      </TouchableOpacity>

      <FlatList
        data={fileList}
        keyExtractor={(item) => item.path}
        renderItem={renderItem}
        style={styles.fileList}
      />
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
    backgroundColor: '#455A64',
  },
  title: {
    fontSize: 24,
    fontWeight: 'bold',
    color: '#FFFFFF',
    marginBottom: 20,
  },
  timer: {
    fontSize: 18,
    color: '#FFFFFF',
    marginBottom: 20,
  },
  buttonGroup: {
    flexDirection: 'row',
    justifyContent: 'space-around',
    marginVertical: 10,
  },
  button: {
    backgroundColor: '#607D8B',
    padding: 10,
    margin: 5,
    borderRadius: 5,
  },
  buttonText: {
    color: '#FFFFFF',
    fontWeight: 'bold',
  },
  divider: {
    width: '90%',
    height: 1,
    backgroundColor: '#FFFFFF',
    marginVertical: 20,
  },
  fileList: {
    width: '90%',
    marginTop: 20,
  },
});

export default App;
