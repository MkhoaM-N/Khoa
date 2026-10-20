# Khoa
////-----App.tsx-----////
import React, { useState, useEffect } from 'react';
import {
  View,
  Text,
  FlatList,
  ActivityIndicator,
  Switch,
  RefreshControl,
  Alert,
  StyleSheet,
} from 'react-native';
import { SafeAreaProvider, SafeAreaView } from 'react-native-safe-area-context';
import MovieCard, { Movie } from './components/MovieCard';

// API(mockAPI)
const API_URL = 'https://6abba219b2118ed7abb919de.mockapi.io/movies';

export default function App() {
  const [movies, setMovies] = useState<Movie[]>([]);
  const [loading, setLoading] = useState(true);
  const [refreshing, setRefreshing] = useState(false);
  const [isTile, setIsTile] = useState(false); // Câu 5a

  // Câu 2: gọi API GET
  const getMovies = async () => {
    try {
      const res = await fetch(API_URL);
      const data = await res.json();
      setMovies(data);
    } catch (error) {
      console.log(error);
    }
  };

  // Lần đầu mở app: hiện ActivityIndicator trong lúc tải
  const loadData = async () => {
    setLoading(true);
    await getMovies();
    setLoading(false);
  };

  useEffect(() => {
    loadData();
  }, []);

  // Câu 6: kéo xuống làm mới, gọi lại API GET
  const onRefresh = async () => {
    setRefreshing(true);
    await getMovies();
    setRefreshing(false);
  };

  // Câu 3c: nhấn thẻ thì hiện Alert tên phim
  const handleSelect = (id: string) => {
    const movie = movies.find((m) => String(m.id) === id);
    if (movie) {
      Alert.alert('Tên phim', movie.title);
    }
  };

  const numColumns = isTile ? 2 : 1; // Câu 5b

  return (
    // Câu 1a: bọc toàn bộ app trong SafeAreaProvider
    <SafeAreaProvider>
      {/* Câu 1b: SafeAreaView + View */}
      <SafeAreaView style={styles.container}>
        <View style={styles.header}>
          {/* Câu 1c: tiêu đề */}
          <Text style={styles.title}>Movie App</Text>

          {/* Câu 5a: Switch "Dạng lưới" */}
          <View style={styles.switchRow}>
            <Text>Dạng lưới</Text>
            <Switch value={isTile} onValueChange={setIsTile} />
          </View>
        </View>

        {loading ? (
          // Câu 2c
          <ActivityIndicator size="large" color="blue" />
        ) : (
          <FlatList
            key={String(numColumns)} // Câu 5c
            data={movies}
            keyExtractor={(item) => String(item.id)}
            numColumns={numColumns} // Câu 5b
            columnWrapperStyle={isTile ? styles.column : undefined} // Câu 5d
            contentContainerStyle={styles.list}
            renderItem={({item}) => (
              <MovieCard
                movie={item}
                layout={isTile ? 'tile' : 'row'}
                onSelect={handleSelect}
              />
            )}
            refreshControl={
              <RefreshControl refreshing={refreshing} onRefresh={onRefresh} />
            }
          />
        )}
      </SafeAreaView>
    </SafeAreaProvider>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#eee',
  },
  header: {
    padding: 12,
    backgroundColor: '#fff',
  },
  title: {
    fontSize: 24,
    fontWeight: 'bold',
    textAlign: 'center',
  },
  switchRow: {
    flexDirection: 'row',
    alignItems: 'center',
    justifyContent: 'flex-end',
    marginTop: 6,
  },
  list: {
    padding: 10,
  },
  // Câu 5d: 2 thẻ cách đều, thẻ lẻ cuối vẫn giữ 48%
  column: {
    justifyContent: 'space-between',
  },
});

                /---------/Component-moviecard/------------/


import React from 'react';
import { View, Text, Image, TouchableOpacity, StyleSheet } from 'react-native';

// Câu 3a: kiểu dữ liệu Movie
export type Movie = {
  id: string;
  title: string;
  genre: string;
  year: number;
  rating: number;
  poster: string;
  isShowing: boolean;
};

// Câu 3a: kiểu props của MovieCard
export type MovieCardProps = {
  movie: Movie;
  layout?: 'row' | 'tile';
  onSelect: (id: string) => void;
};

function MovieCard({ movie, layout = 'row', onSelect }: MovieCardProps) {
  const isTile = layout === 'tile';

  // Câu 3b: điểm dạng "⭐ 8.5" (1 chữ số thập phân)
  const ratingText = '⭐ ' + Number(movie.rating).toFixed(1);

  return (
    // Câu 3c: bọc ngoài bằng TouchableOpacity
    <TouchableOpacity
      testID="movie-card"
      style={[styles.card, isTile && styles.cardTile]}
      onPress={() => onSelect(movie.id)}
    >
      <View>
        <Image
          source={{ uri: movie.poster }}
          style={[styles.poster, isTile && styles.posterTile]}
        />
        {/* Câu 4b: nhãn ⭐ đè lên góc poster (chỉ ở dạng tile) */}
        {isTile && (
          <View style={styles.badge}>
            <Text style={styles.badgeText}>{ratingText}</Text>
          </View>
        )}
      </View>

      <View style={[styles.info, isTile && styles.infoTile]}>
        {/* Câu 4b: tile thì tên phim chỉ 1 dòng */}
        <Text style={styles.title} numberOfLines={isTile ? 1 : 2}>
          {movie.title}
        </Text>

        {/* Câu 4a: row thì hiện thể loại, năm, điểm. Tile thì ẩn */}
        {!isTile && <Text style={styles.sub}>{movie.genre}</Text>}
        {!isTile && <Text style={styles.sub}>Năm: {movie.year}</Text>}
        {!isTile && <Text style={styles.rating}>{ratingText}</Text>}

        {/* Câu 3b: trạng thái */}
        <Text>{movie.isShowing ? '✅' : '❌'}</Text>
      </View>
    </TouchableOpacity>
  );
}

// Câu 4c: StyleSheet.create, kết hợp style theo mảng
const styles = StyleSheet.create({
  card: {
    flexDirection: 'row',
    backgroundColor: '#fff',
    borderRadius: 8,
    padding: 8,
    marginBottom: 10,
  },
  cardTile: {
    flexDirection: 'column',
    width: '48%',
  },
  poster: {
    width: 70,
    height: 100,
    borderRadius: 6,
  },
  posterTile: {
    width: '100%',
    height: undefined,
    aspectRatio: 2 / 3,
  },
  badge: {
    position: 'absolute',
    top: 6,
    left: 6,
    backgroundColor: 'rgba(0,0,0,0.6)',
    paddingHorizontal: 6,
    paddingVertical: 2,
    borderRadius: 4,
  },
  badgeText: {
    color: '#fff',
    fontSize: 12,
  },
  info: {
    flex: 1,
    marginLeft: 10,
  },
  infoTile: {
    marginLeft: 0,
    marginTop: 6,
  },
  title: {
    fontSize: 16,
    fontWeight: 'bold',
  },
  sub: {
    color: '#666',
  },
  rating: {
    marginVertical: 2,
  },
});

// Câu 3d: export qua React.memo
export default React.memo(MovieCard);

               /---------/ __tests__/MovieCard.test.tsx/------------/
import React from 'react';
import { View, Text, Image, TouchableOpacity, StyleSheet } from 'react-native';

// Câu 3a: kiểu dữ liệu Movie
export type Movie = {
  id: string;
  title: string;
  genre: string;
  year: number;
  rating: number;
  poster: string;
  isShowing: boolean;
};

// Câu 3a: kiểu props của MovieCard
export type MovieCardProps = {
  movie: Movie;
  layout?: 'row' | 'tile';
  onSelect: (id: string) => void;
};

function MovieCard({ movie, layout = 'row', onSelect }: MovieCardProps) {
  const isTile = layout === 'tile';

  // Câu 3b: điểm dạng "⭐ 8.5" (1 chữ số thập phân)
  const ratingText = '⭐ ' + Number(movie.rating).toFixed(1);

  return (
    // Câu 3c: bọc ngoài bằng TouchableOpacity
    <TouchableOpacity
      testID="movie-card"
      style={[styles.card, isTile && styles.cardTile]}
      onPress={() => onSelect(movie.id)}
    >
      <View>
        <Image
          source={{ uri: movie.poster }}
          style={[styles.poster, isTile && styles.posterTile]}
        />
        {/* Câu 4b: nhãn ⭐ đè lên góc poster (chỉ ở dạng tile) */}
        {isTile && (
          <View style={styles.badge}>
            <Text style={styles.badgeText}>{ratingText}</Text>
          </View>
        )}
      </View>

      <View style={[styles.info, isTile && styles.infoTile]}>
        {/* Câu 4b: tile thì tên phim chỉ 1 dòng */}
        <Text style={styles.title} numberOfLines={isTile ? 1 : 2}>
          {movie.title}
        </Text>

        {/* Câu 4a: row thì hiện thể loại, năm, điểm. Tile thì ẩn */}
        {!isTile && <Text style={styles.sub}>{movie.genre}</Text>}
        {!isTile && <Text style={styles.sub}>Năm: {movie.year}</Text>}
        {!isTile && <Text style={styles.rating}>{ratingText}</Text>}

        {/* Câu 3b: trạng thái */}
        <Text>{movie.isShowing ? '✅' : '❌'}</Text>
      </View>
    </TouchableOpacity>
  );
}

// Câu 4c: StyleSheet.create, kết hợp style theo mảng
const styles = StyleSheet.create({
  card: {
    flexDirection: 'row',
    backgroundColor: '#fff',
    borderRadius: 8,
    padding: 8,
    marginBottom: 10,
  },
  cardTile: {
    flexDirection: 'column',
    width: '48%',
  },
  poster: {
    width: 70,
    height: 100,
    borderRadius: 6,
  },
  posterTile: {
    width: '100%',
    height: undefined,
    aspectRatio: 2 / 3,
  },
  badge: {
    position: 'absolute',
    top: 6,
    left: 6,
    backgroundColor: 'rgba(0,0,0,0.6)',
    paddingHorizontal: 6,
    paddingVertical: 2,
    borderRadius: 4,
  },
  badgeText: {
    color: '#fff',
    fontSize: 12,
  },
  info: {
    flex: 1,
    marginLeft: 10,
  },
  infoTile: {
    marginLeft: 0,
    marginTop: 6,
  },
  title: {
    fontSize: 16,
    fontWeight: 'bold',
  },
  sub: {
    color: '#666',
  },
  rating: {
    marginVertical: 2,
  },
});

// Câu 3d: export qua React.memo
export default React.memo(MovieCard);
